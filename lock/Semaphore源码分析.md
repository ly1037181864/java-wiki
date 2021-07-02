###Semaphore源码分析
```text
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
```text
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())//线程中断，则直接抛异常
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)//尝试获取锁
        doAcquireSharedInterruptibly(arg);
}
```
```text
protected int tryAcquireShared(int acquires) {
    for (;;) {
        if (hasQueuedPredecessors())//如果当前有线程在等待获取锁，且非当前线程，那么直接返回-1
            return -1;
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;//多个线程尝试获取锁，直接减1操作，如果线程数超过计数器则返回负数，意味着获取锁失败
    }
}
```
```text
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.SHARED);//构建同步队列
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);//如果计数器复位，且当前head节点的下一个节点就是当前线程，那么获取锁成功
                if (r >= 0) {
                    setHeadAndPropagate(node, r);//会替换head节点，并唤醒后续节点
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            //如果尝试获取锁失败后会挂起线程，再次被唤醒后会执行上述的逻辑
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
```text
public void release() {
    sync.releaseShared(1);
}
```
```text
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {//尝试释放锁
        doReleaseShared();
        return true;
    }
    return false;
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
                unparkSuccessor(h);//唤醒head节点的下一个节点
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
private boolean findNodeFromTail(Node node) {
    Node t = tail;
    for (;;) {
        if (t == node)
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}
```