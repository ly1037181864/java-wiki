###CountDownLatch源码分析
```text
public void countDown() {
    sync.releaseShared(1);//计数器减1
}
```
```text
public final boolean releaseShared(int arg) {
    //判断计数器是否归0 ，如果计数器归0则进行后续的逻辑操作，否则返回false
    if (tryReleaseShared(arg)) {
        //计数器归0后的操作
        doReleaseShared();
        return true;
    }
    return false;
}
```
```text
protected boolean tryReleaseShared(int releases) {
    // Decrement count; signal when transition to zero
    for (;;) {
        int c = getState();//获取计数器
        if (c == 0) //如果计数器已经归
            return false;//直接返回false
        int nextc = c-1;//计数器减1
        if (compareAndSetState(c, nextc))
            return nextc == 0;//如果计数器归0返回true，否则返回false
    }
}
```
```text
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);//所有计数器归0后，唤醒被挂起的线程
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

```text
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
```text
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())//如果收到过线程中断，那么直接抛出异常
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)//如果计数器归0 则返回1 否则返回-1
        doAcquireSharedInterruptibly(arg);//意味着计数器还没有归0
}
```
```text
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;//计数器归0 返回1 否则返回-1
}
```
```text
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.SHARED);//构建共享型的同步队列
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);//如果此时计数器仍然还没有归0，那么直接挂起当前线程，否则获得锁成功，执行后续的逻辑操作，即不再阻塞
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```