###Condition源码分析
```text
public final void await() throws InterruptedException {
    if (Thread.interrupted())//如果线程中断，则直接抛异常
        throw new InterruptedException();
    Node node = addConditionWaiter();
    long savedState = fullyRelease(node);//释放锁
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {//如果当前节点不再同步队列中
        LockSupport.park(this);//挂起当前线程
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)//如果期间收到过中断请求，则跳出循环，否则继续循环挂起，直到当前note节点进入同步队列中
            break;
    }
    //尝试获取锁
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```
```text
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    if (t != null && t.waitStatus != Node.CONDITION) {//如果等待队列尾节点不为等待状态，则需要清除该节点
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    Node node = new Node(Thread.currentThread(), Node.CONDITION);//构建等待状态节点并加入等待队列末尾
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```
```text
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null;//当前节点的上一个节点
    while (t != null) {
        Node next = t.nextWaiter;//当前节点的下一个节点
        if (t.waitStatus != Node.CONDITION) {//如果当前节点不等于等待状态
            t.nextWaiter = null;//将其下一个节点置空
            if (trail == null)//如果当前节点的上一个节点为空，意味这是第一次进入循环
                firstWaiter = next;//则直接将头节点指向当前节点的下一个节点，也即队列中剔除当前节点
            else
                trail.nextWaiter = next;//如果当前当前节点的前一个节点不为空，那么直接将当前节点的前一个节点指向当前节点的下一个节点，目的也是为了从队列中剔除当前节点
            if (next == null)//如果当前节点的下一个节点为空，则说明当前节点是尾节点
                lastWaiter = trail;//则直接将尾节点指向当前节点的上一个节点，目的也是为了从队列中剔除当前节点
        }
        else
            trail = t;//将当前节点赋值给trail，作为下一次循环中当前节点的前一个节点
        t = next;
    }
}
```
```text
final boolean isOnSyncQueue(Node node) {
    //如果当前节点是等待状态或者当前节点的前一个节点为空则表示不再同步队列，因为等待队列是单向队列，当前节点的前一个节点当然为空
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
     //到这里如果当前节点的下一个节点也不为空，且不是等待状态，那么当前节点一定是在同步队列中
    if (node.next != null) // If has successor, it must be on queue
        return true;
     //如果当前note节点的前一个节点不为空，又不是等待状态，那么可能的情况就是在同步队列中挂起线程前加入同步队列进行CAS替换时失败了，所以这个时候让仍然需要去再次尝试遍历判断同步是否存在该节点
    return findNodeFromTail(node);
}
```
```text
public final void signal() {
    //唤醒线程不是获取锁的线程抛异常
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;//等待队首唤醒
    if (first != null)
        doSignal(first);
}
```
```text
private void doSignal(Node first) {
    do {
        //获取等待队列的下一个节点，如果等待队列头节点的下一个节点为空
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;//清空尾节点
        first.nextWaiter = null;//清空头节点的下一个节点
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
```
```text
final boolean transferForSignal(Node node) {
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    //将等待队列中的note移动到同步队列，把当前note移动到同步队列的目的是为wait方法在循环判断当前note节点是否在同步队列时，条件为真
    Node p = enq(node);
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);//唤醒该线程即唤醒在wait中阻塞的线程
    return true;
}
```