TLDR:

I wrote a library that allows you to do this:

    from fluidIter import FluidIterable
    fSomeList = FluidIterable(someList)  
    for tup in fSomeList:
        if determine(tup):
            # remove 'tup' without breaking the iteration
            fSomeList.remove(tup)
            # tup has also been removed from 'someList'
            # as well as 'fSomeList'

It's best to use a list comprehension if possible, but for some algorithms it isn't that straight forward.

Should work on all mutable sequences not just lists.

---

Full answer:

Edit: The last code example in this answer gives a use case for ***why*** you might sometimes want to modify a list in place rather than use a list comprehension. The first part of the answers serves as tutorial of ***how*** an array can be modified in place.

The solution follows on from [this][1] answer (for a related question) from senderle. Which explains how the the array index is updated while iterating through a list that has been modified. The solution below is designed to correctly track the array index even if the list is modified.

Download `fluidIter.py` from [here][2] `https://github.com/alanbacon/FluidIterator`, it is just a single file so no need to install git. There is no installer so you will need to make sure that the file is in the python path your self. The code has been written for python 3 and is untested on python 2.

    from fluidIter import FluidIterable
    l = [0,1,2,3,4,5,6,7,8]  
    fluidL = FluidIterable(l)                       
    for i in fluidL:
        print('initial state of list on this iteration: ' + str(fluidL)) 
        print('current iteration value: ' + str(i))
        print('popped value: ' + str(fluidL.pop(2)))
        print(' ')

    print('Final List Value: ' + str(l))

This will produce the following output:

    initial state of list on this iteration: [0, 1, 2, 3, 4, 5, 6, 7, 8]
    current iteration value: 0
    popped value: 2
 
    initial state of list on this iteration: [0, 1, 3, 4, 5, 6, 7, 8]
    current iteration value: 1
    popped value: 3
 
    initial state of list on this iteration: [0, 1, 4, 5, 6, 7, 8]
    current iteration value: 4
    popped value: 4
  
    initial state of list on this iteration: [0, 1, 5, 6, 7, 8]
    current iteration value: 5
    popped value: 5
 
    initial state of list on this iteration: [0, 1, 6, 7, 8]
    current iteration value: 6
    popped value: 6
 
    initial state of list on this iteration: [0, 1, 7, 8]
    current iteration value: 7
    popped value: 7
 
    initial state of list on this iteration: [0, 1, 8]
    current iteration value: 8
    popped value: 8

    Final List Value: [0, 1]

Above we have used the `pop` method on the fluid list object. Other common iterable methods are also implemented such as `del fluidL[i]`, `.remove`, `.insert`, `.append`, `.extend`. The list can also be modified using slices (`sort` and `reverse` methods are not implemented).

The only condition is that you must only modify the list in place, if at any point `fluidL` or `l` were reassigned to a different list object the code would not work. The original `fluidL` object would still be used by the for loop but would become out of scope for us to modify.

i.e.

    fluidL[2] = 'a'   # is OK
    fluidL = [0, 1, 'a', 3, 4, 5, 6, 7, 8]  # is not OK

If we want to access the current index value of the list we cannot use enumerate, as this only counts how many times the for loop has run. Instead we will use the iterator object directly.

    fluidArr = FluidIterable([0,1,2,3])
    # get iterator first so can query the current index
    fluidArrIter = fluidArr.__iter__()
    for i, v in enumerate(fluidArrIter):
        print('enum: ', i)
        print('current val: ', v)
        print('current ind: ', fluidArrIter.currentIndex)
        print(fluidArr)
        fluidArr.insert(0,'a')
        print(' ')
        
    print('Final List Value: ' + str(fluidArr))

This will output the following:

    enum:  0
    current val:  0
    current ind:  0
    [0, 1, 2, 3]
     
    enum:  1
    current val:  1
    current ind:  2
    ['a', 0, 1, 2, 3]
     
    enum:  2
    current val:  2
    current ind:  4
    ['a', 'a', 0, 1, 2, 3]
     
    enum:  3
    current val:  3
    current ind:  6
    ['a', 'a', 'a', 0, 1, 2, 3]
     
    Final List Value: ['a', 'a', 'a', 'a', 0, 1, 2, 3]

The `FluidIterable` class just provides a wrapper for the original list object. The original object can be accessed as a property of the fluid object like so:

    originalList = fluidArr.fixedIterable

More examples / tests can be found in the `if __name__ is "__main__":` section at the bottom of `fluidIter.py`. These are worth looking at because they explain what happens in various situations. Such as: Replacing a large sections of the list using a slice. Or using (and modifying) the same iterable in nested for loops.

As I stated to start with: this is a complicated solution that will hurt the readability of your code and make it more difficult to debug. Therefore other solutions such as the list comprehensions mentioned in David Raznick's [answer][3] should be considered first. That being said, I have found times where this class has been useful to me and has been easier to use than keeping track of the indices of elements that need deleting.

------------------------------------------------------------------------

Edit: As mentioned in the comments, this answer does not really present a problem for which this approach provides a solution. I will try to address that here:

List comprehensions provide a way to generate a new list but these approaches tend to look at each element in isolation rather than the current state of the list as a whole.

i.e.

    newList = [i for i in oldList if testFunc(i)]

But what if the result of the `testFunc` depends on the elements that have been added to `newList` already? Or the elements still in `oldList` that might be added next? There might still be a way to use a list comprehension but it will begin to lose it's elegance, and for me it feels easier to modify a list in place.

The code below is one example of an algorithm that suffers from the above problem. The algorithm will reduce a list so that no element is a multiple of any other element.

    randInts = [70, 20, 61, 80, 54, 18, 7, 18, 55, 9]
    fRandInts = FluidIterable(randInts)
    fRandIntsIter = fRandInts.__iter__()
    # for each value in the list (outer loop)
    # test against every other value in the list (inner loop)
    for i in fRandIntsIter:
        print(' ')
        print('outer val: ', i)
        innerIntsIter = fRandInts.__iter__()
        for j in innerIntsIter:
            innerIndex = innerIntsIter.currentIndex
            # skip the element that the outloop is currently on
            # because we don't want to test a value against itself
            if not innerIndex == fRandIntsIter.currentIndex:
                # if the test element, j, is a multiple 
                # of the reference element, i, then remove 'j'
                if j%i == 0:
                    print('remove val: ', j)
                    # remove element in place, without breaking the
                    # iteration of either loop
                    del fRandInts[innerIndex]
                # end if multiple, then remove
            # end if not the same value as outer loop
        # end inner loop
    # end outerloop
    
    print('')
    print('final list: ', randInts)

The output and the final reduced list are shown below

    outer val:  70
    
    outer val:  20
    remove val:  80
     
    outer val:  61
     
    outer val:  54
     
    outer val:  18
    remove val:  54
    remove val:  18
     
    outer val:  7
    remove val:  70
     
    outer val:  55
     
    outer val:  9
    remove val:  18
    
    final list:  [20, 61, 7, 55, 9]


  [1]: http://stackoverflow.com/a/6260097/4451578
  [2]: https://github.com/alanbacon/FluidIterator
  [3]: http://stackoverflow.com/a/1207461/4451578
