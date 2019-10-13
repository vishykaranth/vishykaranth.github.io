## PriorityBlockingQueue
    /*
     * The implementation uses an array-based binary heap, with public
     * operations protected with a single lock. However, allocation
     * during resizing uses a simple spinlock (used only while not
     * holding main lock) in order to allow takes to operate
     * concurrently with allocation.  This avoids repeated
     * postponement of waiting consumers and consequent element
     * build-up. The need to back away from lock during allocation
     * makes it impossible to simply wrap delegated
     * java.util.PriorityQueue operations within a lock, as was done
     * in a previous version of this class. To maintain
     * interoperability, a plain PriorityQueue is still used during
     * serialization, which maintains compatibility at the expense of
     * transiently doubling overhead.
     */
     
## LinkedBlockingQueue
    /*
     * A variant of the "two lock queue" algorithm.  The putLock gates
     * entry to put (and offer), and has an associated condition for
     * waiting puts.  Similarly for the takeLock.  The "count" field
     * that they both rely on is maintained as an atomic to avoid
     * needing to get both locks in most cases. Also, to minimize need
     * for puts to get takeLock and vice-versa, cascading notifies are
     * used. When a put notices that it has enabled at least one take,
     * it signals taker. That taker in turn signals others if more
     * items have been entered since the signal. And symmetrically for
     * takes signalling puts. Operations such as remove(Object) and
     * iterators acquire both locks.
     *
     * Visibility between writers and readers is provided as follows:
     *
     * Whenever an element is enqueued, the putLock is acquired and
     * count updated.  A subsequent reader guarantees visibility to the
     * enqueued Node by either acquiring the putLock (via fullyLock)
     * or by acquiring the takeLock, and then reading n = count.get();
     * this gives visibility to the first n items.
     *
     * To implement weakly consistent iterators, it appears we need to
     * keep all Nodes GC-reachable from a predecessor dequeued Node.
     * That would cause two problems:
     * - allow a rogue Iterator to cause unbounded memory retention
     * - cause cross-generational linking of old Nodes to new Nodes if
     *   a Node was tenured while live, which generational GCs have a
     *   hard time dealing with, causing repeated major collections.
     * However, only non-deleted Nodes need to be reachable from
     * dequeued Nodes, and reachability does not necessarily have to
     * be of the kind understood by the GC.  We use the trick of
     * linking a Node that has just been dequeued to itself.  Such a
     * self-link implicitly means to advance to head.next.
     */

## LinkedTransferQueue

    /*
     * *** Overview of Dual Queues with Slack ***
     *
     * Dual Queues, introduced by Scherer and Scott
     * (http://www.cs.rice.edu/~wns1/papers/2004-DISC-DDS.pdf) are
     * (linked) queues in which nodes may represent either data or
     * requests.  When a thread tries to enqueue a data node, but
     * encounters a request node, it instead "matches" and removes it;
     * and vice versa for enqueuing requests. Blocking Dual Queues
     * arrange that threads enqueuing unmatched requests block until
     * other threads provide the match. Dual Synchronous Queues (see
     * Scherer, Lea, & Scott
     * http://www.cs.rochester.edu/u/scott/papers/2009_Scherer_CACM_SSQ.pdf)
     * additionally arrange that threads enqueuing unmatched data also
     * block.  Dual Transfer Queues support all of these modes, as
     * dictated by callers.
     *
     * A FIFO dual queue may be implemented using a variation of the
     * Michael & Scott (M&S) lock-free queue algorithm
     * (http://www.cs.rochester.edu/u/scott/papers/1996_PODC_queues.pdf).
     * It maintains two pointer fields, "head", pointing to a
     * (matched) node that in turn points to the first actual
     * (unmatched) queue node (or null if empty); and "tail" that
     * points to the last node on the queue (or again null if
     * empty). For example, here is a possible queue with four data
     * elements:
     *
     *  head                tail
     *    |                   |
     *    v                   v
     *    M -> U -> U -> U -> U
     *
     * The M&S queue algorithm is known to be prone to scalability and
     * overhead limitations when maintaining (via CAS) these head and
     * tail pointers. This has led to the development of
     * contention-reducing variants such as elimination arrays (see
     * Moir et al http://portal.acm.org/citation.cfm?id=1074013) and
     * optimistic back pointers (see Ladan-Mozes & Shavit
     * http://people.csail.mit.edu/edya/publications/OptimisticFIFOQueue-journal.pdf).
     * However, the nature of dual queues enables a simpler tactic for
     * improving M&S-style implementations when dual-ness is needed.
     *
     * In a dual queue, each node must atomically maintain its match
     * status. While there are other possible variants, we implement
     * this here as: for a data-mode node, matching entails CASing an
     * "item" field from a non-null data value to null upon match, and
     * vice-versa for request nodes, CASing from null to a data
     * value. (Note that the linearization properties of this style of
     * queue are easy to verify -- elements are made available by
     * linking, and unavailable by matching.) Compared to plain M&S
     * queues, this property of dual queues requires one additional
     * successful atomic operation per enq/deq pair. But it also
     * enables lower cost variants of queue maintenance mechanics. (A
     * variation of this idea applies even for non-dual queues that
     * support deletion of interior elements, such as
     * j.u.c.ConcurrentLinkedQueue.)
     *
     * Once a node is matched, its match status can never again
     * change.  We may thus arrange that the linked list of them
     * contain a prefix of zero or more matched nodes, followed by a
     * suffix of zero or more unmatched nodes. (Note that we allow
     * both the prefix and suffix to be zero length, which in turn
     * means that we do not use a dummy header.)  If we were not
     * concerned with either time or space efficiency, we could
     * correctly perform enqueue and dequeue operations by traversing
     * from a pointer to the initial node; CASing the item of the
     * first unmatched node on match and CASing the next field of the
     * trailing node on appends. (Plus some special-casing when
     * initially empty).  While this would be a terrible idea in
     * itself, it does have the benefit of not requiring ANY atomic
     * updates on head/tail fields.
     *
     * We introduce here an approach that lies between the extremes of
     * never versus always updating queue (head and tail) pointers.
     * This offers a tradeoff between sometimes requiring extra
     * traversal steps to locate the first and/or last unmatched
     * nodes, versus the reduced overhead and contention of fewer
     * updates to queue pointers. For example, a possible snapshot of
     * a queue is:
     *
     *  head           tail
     *    |              |
     *    v              v
     *    M -> M -> U -> U -> U -> U
     *
     * The best value for this "slack" (the targeted maximum distance
     * between the value of "head" and the first unmatched node, and
     * similarly for "tail") is an empirical matter. We have found
     * that using very small constants in the range of 1-3 work best
     * over a range of platforms. Larger values introduce increasing
     * costs of cache misses and risks of long traversal chains, while
     * smaller values increase CAS contention and overhead.
     *
     * Dual queues with slack differ from plain M&S dual queues by
     * virtue of only sometimes updating head or tail pointers when
     * matching, appending, or even traversing nodes; in order to
     * maintain a targeted slack.  The idea of "sometimes" may be
     * operationalized in several ways. The simplest is to use a
     * per-operation counter incremented on each traversal step, and
     * to try (via CAS) to update the associated queue pointer
     * whenever the count exceeds a threshold. Another, that requires
     * more overhead, is to use random number generators to update
     * with a given probability per traversal step.
     *
     * In any strategy along these lines, because CASes updating
     * fields may fail, the actual slack may exceed targeted
     * slack. However, they may be retried at any time to maintain
     * targets.  Even when using very small slack values, this
     * approach works well for dual queues because it allows all
     * operations up to the point of matching or appending an item
     * (hence potentially allowing progress by another thread) to be
     * read-only, thus not introducing any further contention. As
     * described below, we implement this by performing slack
     * maintenance retries only after these points.
     *
     * As an accompaniment to such techniques, traversal overhead can
     * be further reduced without increasing contention of head
     * pointer updates: Threads may sometimes shortcut the "next" link
     * path from the current "head" node to be closer to the currently
     * known first unmatched node, and similarly for tail. Again, this
     * may be triggered with using thresholds or randomization.
     *
     * These ideas must be further extended to avoid unbounded amounts
     * of costly-to-reclaim garbage caused by the sequential "next"
     * links of nodes starting at old forgotten head nodes: As first
     * described in detail by Boehm
     * (http://portal.acm.org/citation.cfm?doid=503272.503282) if a GC
     * delays noticing that any arbitrarily old node has become
     * garbage, all newer dead nodes will also be unreclaimed.
     * (Similar issues arise in non-GC environments.)  To cope with
     * this in our implementation, upon CASing to advance the head
     * pointer, we set the "next" link of the previous head to point
     * only to itself; thus limiting the length of connected dead lists.
     * (We also take similar care to wipe out possibly garbage
     * retaining values held in other Node fields.)  However, doing so
     * adds some further complexity to traversal: If any "next"
     * pointer links to itself, it indicates that the current thread
     * has lagged behind a head-update, and so the traversal must
     * continue from the "head".  Traversals trying to find the
     * current tail starting from "tail" may also encounter
     * self-links, in which case they also continue at "head".
     *
     * It is tempting in slack-based scheme to not even use CAS for
     * updates (similarly to Ladan-Mozes & Shavit). However, this
     * cannot be done for head updates under the above link-forgetting
     * mechanics because an update may leave head at a detached node.
     * And while direct writes are possible for tail updates, they
     * increase the risk of long retraversals, and hence long garbage
     * chains, which can be much more costly than is worthwhile
     * considering that the cost difference of performing a CAS vs
     * write is smaller when they are not triggered on each operation
     * (especially considering that writes and CASes equally require
     * additional GC bookkeeping ("write barriers") that are sometimes
     * more costly than the writes themselves because of contention).
     *
     * *** Overview of implementation ***
     *
     * We use a threshold-based approach to updates, with a slack
     * threshold of two -- that is, we update head/tail when the
     * current pointer appears to be two or more steps away from the
     * first/last node. The slack value is hard-wired: a path greater
     * than one is naturally implemented by checking equality of
     * traversal pointers except when the list has only one element,
     * in which case we keep slack threshold at one. Avoiding tracking
     * explicit counts across method calls slightly simplifies an
     * already-messy implementation. Using randomization would
     * probably work better if there were a low-quality dirt-cheap
     * per-thread one available, but even ThreadLocalRandom is too
     * heavy for these purposes.
     *
     * With such a small slack threshold value, it is not worthwhile
     * to augment this with path short-circuiting (i.e., unsplicing
     * interior nodes) except in the case of cancellation/removal (see
     * below).
     *
     * We allow both the head and tail fields to be null before any
     * nodes are enqueued; initializing upon first append.  This
     * simplifies some other logic, as well as providing more
     * efficient explicit control paths instead of letting JVMs insert
     * implicit NullPointerExceptions when they are null.  While not
     * currently fully implemented, we also leave open the possibility
     * of re-nulling these fields when empty (which is complicated to
     * arrange, for little benefit.)
     *
     * All enqueue/dequeue operations are handled by the single method
     * "xfer" with parameters indicating whether to act as some form
     * of offer, put, poll, take, or transfer (each possibly with
     * timeout). The relative complexity of using one monolithic
     * method outweighs the code bulk and maintenance problems of
     * using separate methods for each case.
     *
     * Operation consists of up to three phases. The first is
     * implemented within method xfer, the second in tryAppend, and
     * the third in method awaitMatch.
     *
     * 1. Try to match an existing node
     *
     *    Starting at head, skip already-matched nodes until finding
     *    an unmatched node of opposite mode, if one exists, in which
     *    case matching it and returning, also if necessary updating
     *    head to one past the matched node (or the node itself if the
     *    list has no other unmatched nodes). If the CAS misses, then
     *    a loop retries advancing head by two steps until either
     *    success or the slack is at most two. By requiring that each
     *    attempt advances head by two (if applicable), we ensure that
     *    the slack does not grow without bound. Traversals also check
     *    if the initial head is now off-list, in which case they
     *    start at the new head.
     *
     *    If no candidates are found and the call was untimed
     *    poll/offer, (argument "how" is NOW) return.
     *
     * 2. Try to append a new node (method tryAppend)
     *
     *    Starting at current tail pointer, find the actual last node
     *    and try to append a new node (or if head was null, establish
     *    the first node). Nodes can be appended only if their
     *    predecessors are either already matched or are of the same
     *    mode. If we detect otherwise, then a new node with opposite
     *    mode must have been appended during traversal, so we must
     *    restart at phase 1. The traversal and update steps are
     *    otherwise similar to phase 1: Retrying upon CAS misses and
     *    checking for staleness.  In particular, if a self-link is
     *    encountered, then we can safely jump to a node on the list
     *    by continuing the traversal at current head.
     *
     *    On successful append, if the call was ASYNC, return.
     *
     * 3. Await match or cancellation (method awaitMatch)
     *
     *    Wait for another thread to match node; instead cancelling if
     *    the current thread was interrupted or the wait timed out. On
     *    multiprocessors, we use front-of-queue spinning: If a node
     *    appears to be the first unmatched node in the queue, it
     *    spins a bit before blocking. In either case, before blocking
     *    it tries to unsplice any nodes between the current "head"
     *    and the first unmatched node.
     *
     *    Front-of-queue spinning vastly improves performance of
     *    heavily contended queues. And so long as it is relatively
     *    brief and "quiet", spinning does not much impact performance
     *    of less-contended queues.  During spins threads check their
     *    interrupt status and generate a thread-local random number
     *    to decide to occasionally perform a Thread.yield. While
     *    yield has underdefined specs, we assume that it might help,
     *    and will not hurt, in limiting impact of spinning on busy
     *    systems.  We also use smaller (1/2) spins for nodes that are
     *    not known to be front but whose predecessors have not
     *    blocked -- these "chained" spins avoid artifacts of
     *    front-of-queue rules which otherwise lead to alternating
     *    nodes spinning vs blocking. Further, front threads that
     *    represent phase changes (from data to request node or vice
     *    versa) compared to their predecessors receive additional
     *    chained spins, reflecting longer paths typically required to
     *    unblock threads during phase changes.
     *
     *
     * ** Unlinking removed interior nodes **
     *
     * In addition to minimizing garbage retention via self-linking
     * described above, we also unlink removed interior nodes. These
     * may arise due to timed out or interrupted waits, or calls to
     * remove(x) or Iterator.remove.  Normally, given a node that was
     * at one time known to be the predecessor of some node s that is
     * to be removed, we can unsplice s by CASing the next field of
     * its predecessor if it still points to s (otherwise s must
     * already have been removed or is now offlist). But there are two
     * situations in which we cannot guarantee to make node s
     * unreachable in this way: (1) If s is the trailing node of list
     * (i.e., with null next), then it is pinned as the target node
     * for appends, so can only be removed later after other nodes are
     * appended. (2) We cannot necessarily unlink s given a
     * predecessor node that is matched (including the case of being
     * cancelled): the predecessor may already be unspliced, in which
     * case some previous reachable node may still point to s.
     * (For further explanation see Herlihy & Shavit "The Art of
     * Multiprocessor Programming" chapter 9).  Although, in both
     * cases, we can rule out the need for further action if either s
     * or its predecessor are (or can be made to be) at, or fall off
     * from, the head of list.
     *
     * Without taking these into account, it would be possible for an
     * unbounded number of supposedly removed nodes to remain
     * reachable.  Situations leading to such buildup are uncommon but
     * can occur in practice; for example when a series of short timed
     * calls to poll repeatedly time out but never otherwise fall off
     * the list because of an untimed call to take at the front of the
     * queue.
     *
     * When these cases arise, rather than always retraversing the
     * entire list to find an actual predecessor to unlink (which
     * won't help for case (1) anyway), we record a conservative
     * estimate of possible unsplice failures (in "sweepVotes").
     * We trigger a full sweep when the estimate exceeds a threshold
     * ("SWEEP_THRESHOLD") indicating the maximum number of estimated
     * removal failures to tolerate before sweeping through, unlinking
     * cancelled nodes that were not unlinked upon initial removal.
     * We perform sweeps by the thread hitting threshold (rather than
     * background threads or by spreading work to other threads)
     * because in the main contexts in which removal occurs, the
     * caller is already timed-out, cancelled, or performing a
     * potentially O(n) operation (e.g. remove(x)), none of which are
     * time-critical enough to warrant the overhead that alternatives
     * would impose on other threads.
     *
     * Because the sweepVotes estimate is conservative, and because
     * nodes become unlinked "naturally" as they fall off the head of
     * the queue, and because we allow votes to accumulate even while
     * sweeps are in progress, there are typically significantly fewer
     * such nodes than estimated.  Choice of a threshold value
     * balances the likelihood of wasted effort and contention, versus
     * providing a worst-case bound on retention of interior nodes in
     * quiescent queues. The value defined below was chosen
     * empirically to balance these under various timeout scenarios.
     *
     * Note that we cannot self-link unlinked interior nodes during
     * sweeps. However, the associated garbage chains terminate when
     * some successor ultimately falls off the head of the list and is
     * self-linked.
     */
     
     
## DelayQueue

    /**
     * Thread designated to wait for the element at the head of
     * the queue.  This variant of the Leader-Follower pattern
     * (http://www.cs.wustl.edu/~schmidt/POSA/POSA2/) serves to
     * minimize unnecessary timed waiting.  When a thread becomes
     * the leader, it waits only for the next delay to elapse, but
     * other threads await indefinitely.  The leader thread must
     * signal some other thread before returning from take() or
     * poll(...), unless some other thread becomes leader in the
     * interim.  Whenever the head of the queue is replaced with
     * an element with an earlier expiration time, the leader
     * field is invalidated by being reset to null, and some
     * waiting thread, but not necessarily the current leader, is
     * signalled.  So waiting threads must be prepared to acquire
     * and lose leadership while waiting.
     */          