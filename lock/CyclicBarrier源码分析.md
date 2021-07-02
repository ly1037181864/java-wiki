###CyclicBarrier源码分析
```text
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
```
```text
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        final Generation g = generation;//栅栏

        if (g.broken)
            throw new BrokenBarrierException();

        if (Thread.interrupted()) {//如果在await方法前调用过线程中断方法，则会打开栅栏，并唤醒其他等待线程
            breakBarrier();
            throw new InterruptedException();
        }

        int index = --count;//计数器减1
        if (index == 0) {  // tripped 如果计数器归0
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run(); //构造函数有申明栅栏打开后的调用方法时，直接调用构造函数掺入的接口
                ranAction = true;
                nextGeneration();//构建新的栅栏，并唤醒所有等待线程
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        // loop until tripped, broken, interrupted, or timed out
        for (;;) {
            try {
                if (!timed)//如果没有指定等待时间
                    trip.await();//直接阻塞
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);//否则指定阻塞时间等待，不过Condition中，如果等待时间超过指定时间才会挂起线程，
                    //否则只是让CPU放弃时间片，以此来优化线程上下文切换
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    // We're about to finish waiting even if we had not
                    // been interrupted, so this interrupt is deemed to
                    // "belong" to subsequent execution.
                    Thread.currentThread().interrupt();
                }
            }

            //如果栅栏被打开
            if (g.broken)
                throw new BrokenBarrierException();

            //如果栅栏被打开，或者计数器归0
            if (g != generation)
                return index;

            //超时
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```