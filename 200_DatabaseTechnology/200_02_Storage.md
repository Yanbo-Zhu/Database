
# 1 Disk Terminology and Elevator Algorithm

## 1.1 Disk Sections Terminology

Current slides from the lecture contradicts itself because of the illustration from Disk Summary slide (at page: 36). We will remove that slide and update the presentation. Until then, you can ignore the Disk Summary slide. The terminology built between pages [24-35] is still valid, and you can take it as reference.

## 1.2 Elevator (SCAN) Algorithm

You can take the description from Database Systems The Complete Book (2nd edition) in Chapter 13.3.5 (Disk Scheduling and Elevator Algorithm, page 571) as reference. I also added a link to PDF of this book's 2nd edition in the [Book Reference](https://isis.tu-berlin.de/mod/page/view.php?id=2169570 "Book Reference") page.

In essence, the disk head moves toward a direction until there is no more request to process in that direction.

### 1.2.1 Small Example

Below is an illustration to clarify some possible scenarios asked during the exercises:

------(a)---->-----(b)-----(X)-----(c)------

">" is the head, moving towards right to process an existing request "X". "a","b", and "c" are potential spots for new requests to arrive. Following is some potential scenarios:

- A new request arrives at "a", i.e., it arrives at a position that the head already passed. Then this request has to wait until head processes all the request in its current direction and then come back.
- A new request arrives at "b", i.e., it arrives at a position that head is approaching but not passed yet (you have to do the math). Then head stops here and processes the new request, and then proceeds.
- A new request arrives at "c", i.e., it arrives at a position further than head is moving towards (X). Then this new request is scheduled to be processed after the first (X) one, as it is in the same direction.

If we ask a question about this algorithm, you will find all the details and assumptions in order to leave no room for confusion (e.g. start/stop time combined or separate).

If you have any questions, you can either ask here in replies or in-person next week (in Caching exercises).
