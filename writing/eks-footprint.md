# Twelve pods to two

A high-frequency HTTP API sat in EKS behind traffic that was mostly bots. It needed twelve pods at 3 GB to stay up. It still OOM-killed.

The leak was in one endpoint. Automation hit it often enough that the heap never recovered. Heap dumps and allocation profiles found it. Another replica would not have.

After the fix the same traffic ran on one or two pods at 1.5 GB. Spend on that service dropped about 85%. Availability stopped being a memory story.

Order of operations: prove the leak, then right-size. Scaling first just buys a more expensive leak.

No client code here. The shape of the problem is common enough that the sequence is the useful part.
