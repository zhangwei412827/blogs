+++
title= "Quicksort"
date= 2019-10-09T15:06:26+08:00
tags= ["go"]
+++
### golang实现快速排序
``` golang
package main

import (
	"fmt"
	"math/rand"
)

func QuickSort(slc []int) []int {

	middleIndex := len(slc) / 2
	middleItem := slc[middleIndex]

	leftSlc := []int {}
	rightSlc := []int {}

	for i := 0; i < len(slc); i++ {
		if i == middleIndex {
			continue
		}
		if slc[i] <= middleItem {
			leftSlc = append(leftSlc, slc[i])
		}else {
			rightSlc = append(rightSlc, slc[i])
		}
	}


	return append(append(QuickSort(leftSlc), middleItem), QuickSort(rightSlc)...)
}

func main() {
	slc := []int {}

	for i := 0; i < 10; i++ {
		slc = append(slc, rand.Intn(100))
	}

	fmt.Printf("Origin slice is: %v\n", slc)

	newSlc := QuickSort(slc)

	fmt.Printf("Sorted slice is: %v\n", newSlc)
}
```

