# Progressive garbage collection
## Problem with the current version of MQSim
The current version of MQSim performs garbage collection in an atomic manner, it issues reads and writes to copy all the valid pages in the victim block at once.
## The new extended MQSim
The new extended version of MQSim supports progressive garbage collection. The key idea of the PGC is to memoize the type of the last execution. By performing GC and User IO in an alternative manner,  garbage collection tasks are separated into subtasks and user requests can be executed in between. This hugely improves the latency and performance, as the user requests don't have to wait until all garbage collection requests are finished.

## What has been modified, why and where

### Transaction Scheduling Unit
Found in src/ssd/TSU_Priority_OutOfOrder:

Added one key variable to constructors.

Found in src/ssd/TSU_Priority_OutOfOrder:

Traditionally, MQSim issues garbage collection once the threshold is reached. All garbage collection requests are then pushed back into queues and the transaction scheduling unit issues the garbage collection requests one by one. The user requests can only be performed, once all of the garbage collection requests are finished, causing severe latency issues. The progressive garbage collection policy was chosen to overcome this problem. For the read and write transactions, if the garbage collection is not in urgent mode, the transaction scheduling unit then releases garbage collection and user requests one after another, meaning that some user requests are completed ahead of the original version of MQSim. In this way, the average as well as maximum latency can be improved.

### Garbage Collection and Wear Leveling
Found in src/ssd/GC_and_WL_Unit_Page_Level:

To avoid data races, the victim blocks of garbage collection should not be accessed by users. Also, the pages with ongoing erase operations should not be selected as victim blocks for garbage collection.

## Note:
Search for the PGC tag to the modified parts of the code that relate to the PGC scheme.
