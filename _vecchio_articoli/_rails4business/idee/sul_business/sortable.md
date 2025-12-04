---
layout: default
title: Sortable
parent: Sul business
grand_parent:  Idee
nav_fold: true
has_children: true
nav_order: 1
---

https://github.com/SortableJS/Sortable

## FlowPulse Sortable
https://sortablejs.github.io/Sortable/

Nested Sortables Example
NOTE: When using nested Sortables with animation, it is recommended that the fallbackOnBody option is set to true.
It is also always recommended that either the invertSwap option is set to true, or the swapThreshold option is lower than the default value of 1 (eg 0.65).

Item 1.1
Item 2.2
Item 3.2
Item 3.1
Item 3.3
Item 3.4
Item 2.3
Item 2.4
Item 1.2
Item 2.1
Item 2.1
Item 1.3
Item 1.4
Item 2.2
Item 2.3
Item 2.4
Item 1.5
// Loop through each nested sortable element
for (var i = 0; i < nestedSortables.length; i++) {
	new Sortable(nestedSortables[i], {
		group: 'nested',
		animation: 150,
		fallbackOnBody: true,
		swapThreshold: 0.65
	});
}
Plugins
