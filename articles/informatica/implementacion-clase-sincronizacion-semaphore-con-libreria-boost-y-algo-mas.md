# Implementación de la clase `Semaphore` con la librería `boost` (y alguna cosa más)

Una de las primitivas clásicas de la sincronización de [procesos](http://es.wikipedia.org/wiki/Proceso_(inform%C3%A1tica)) e [hilos](http://es.wikipedia.org/wiki/Hilo_de_ejecuci%C3%B3n) —hablando de informática— es el [semáforo](http://es.wikipedia.org/wiki/Sem%C3%A1foro_(programaci%C3%B3n)). La diferencia entre este y el *[mutex](http://es.wikipedia.org/wiki/Mutex)* es que el semáforo se puede bloquear y liberar desde diferentes hilos/procesos mientras que el *mutex* no; es para proteger una zona de código (el acceso a variables por un sólo hilo).

La librería [`boost`](http://www.boost.org/) —una de las mejores librerías generalistas para C++ que hay, por no decir la mejor— no hay un semáforo de este tipo propiamente dicho, sino que se usan [variables de condición](http://en.wikipedia.org/wiki/Condition_variable) para implementarlo; o semáforos con nombre, aunque estos son más específicos para sincronización entre procesos en lugar de entre hilos. Esta implementación se basa mucho en el estándar [POSIX](http://es.wikipedia.org/wiki/POSIX) de hilos (`pthreads`) pero orientándolo a objetos.

Es por esto que, para no tener que usar un variable de condición, un `mutex` y un contador cada vez que se quiere implementar un semáforo, se puede usar esta clase:

~~~~{.cpp}
#include <boost/thread.hpp>
#include <boost/thread/condition_variable.hpp>
#include <boost/thread/mutex.hpp>

class Semaphore {
    private:
        mutable boost::mutex fMutex;
        boost::condition_variable fCondVar;
        unsigned long fCounter;
        
    public:
        Semaphore(unsigned long counter)
          : fMutex(),
            fCondVar(),
            fCounter(counter)
        {
        }

        void post() {
            boost::mutex::scoped_lock lock(fMutex);
            ++fCounter;
            fCondVar.notify_one();
        }

        void wait() {
            boost::mutex::scoped_lock lock(fMutex);
            while(fCounter <= 0)
                fCondVar.wait(lock);
            --fCounter;
        }
        
        bool isLocked() const {
            return fCounter == 0;
        }
};
~~~~

Un buen ejemplo de uso de un semáforo es implementar una cola de datos usada para el famoso [problema del productor/consumidor](http://en.wikipedia.org/wiki/Producer-consumer_problem), es decir, tenemos dos procesos, uno que produce datos y los mete en la cola, y otro que los consume sacándolos de dicha cola teniendo en cuenta que, cuando la cola está vacía, el consumidor está bloqueado esperando a que haya más valores.

Para ello, la implementación de la cola podría ser la siguiente:

~~~~{.cpp}
#include <deque.hpp>

using std::deque;

template <typename T>
class Queue {
    private:
        deque<T> fQueue;
        mutable boost::mutex fAccessLock;
        mutable Semaphore fSem;
        
    public:
        Queue() : fQueue(), fAccessLock(), fSem(0) {}
        
        virtual ~Queue() {}
        
        virtual void push(T elem) {
            fSem.post();
            boost::mutex::scoped_lock lock(fAccessLock);
            fQueue.push_front(elem);
        }
        
        virtual T pop() {
            fSem.wait();
            boost::mutex::scoped_lock lock(fAccessLock);
            T elem = fQueue.back();
            fQueue.pop_back();
            return elem;
        }
};
~~~~

Y el programa de prueba implementando dos funciones, una el productor y otra el consumidor, y usando [hilos de `boost`](http://www.boost.org/doc/libs/1_48_0/doc/html/thread.html) sería:

~~~~{.cpp}
#include <boost/ref.hpp>
#include <boost/date_time.hpp>
#include <iostream>

template <typename T>
void
producer(Queue<T>& q) {
    int counter = 0;
    int times = 5;
    while(!boost::this_thread::interruption_requested() && times-- > 0) {
        for(int i = 0; i < 5; i++) {
            cout << "producer: inserting " << counter << endl;
            q.push(counter++);
        }
        sleep(4);
    }
}

template <typename T>
void
consumer(Queue<T>& q) {
    while(!boost::this_thread::interruption_requested()) {
        cout << "consumer: extracting " << q.pop() << endl;
        boost::this_thread::sleep(boost::posix_time::milliseconds(100));
    }
}

int
main() {
    Queue<int> q(7);
    
    boost::thread p(producer,boost::ref(q));
    sleep(1);
    boost::thread c(consumer,boost::ref(q));
    
    p.join();
    c.join();

    return 0;
}
~~~~

En este ejemplo, el hilo productor inserta cinco números cada 4 segundos (a ráfagas) mientras que el productor saca un número cada 100 milisegundos. Cuando la cola está vacía, gracias al semáforo el hilo consumidor espera hasta que el productor inserte más números, y así sucesivamente.

Pues sí, pues vale, muy bonito, pero ¿para qué se usa esta cola y el productor/consumidor? Pues, en informática, en multitud de software. Por ejemplo, en el reproductor multimedia que usáis todos los días. El hilo productor es el que hilo que lee del disco duro la música insertando los datos leídos en un *buffer* (en la cola), y el consumidor es el hilo que saca los datos de ese *buffer*, los decodifica y los envía a la tarjeta de sonido. Y con el vídeo funciona igual (aparte, claro, de los mecanismos de sincronización entre sonido y vídeo).

Pero en esta implementación hay una cosa que falta: el productor debería parar de insertar elementos cuando la cola esta llena hasta que el consumidor saque algún elemento. Pero esa implementación será para el próximo programa ;) .
