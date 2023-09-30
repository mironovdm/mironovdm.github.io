# Extending the underlying array of a list

List in CPython is not literally a list, but a dynamic array. When items are added, the underlying array is expanded if there is no room in the array for new items. When items are removed, the array is shrunk if there is too much free space.

Let's look at how CPython extends an array. The resizing logic is in the list_resize() function. It has fairly clear comments.

First, if a new computed length of the list will fit into the current allocated space for the array, and the total number of items in the list is more than half the allocated space, then just set the new length for the list, and that's all:

```c
if (allocated >= newsize && newsize >= (allocated >> 1)) {
	assert(self->ob_item != NULL || newsize == 0);
	Py_SET_SIZE(self, newsize);
	return 0;
}
```

Otherwise, reallocation will occur. To expand the array, it's necessary to calculate the new size of the allocated storage. Here's how it happens:

```c
new_allocated = ((size_t)newsize + (newsize >> 3) + 6) & ~(size_t)3;
```

This is a clearer form:

```python
new_allocated = floor(newsize * 1.125 + 6)
new_allocated = new_allocated - new_allocated % 4
```

The new size is increased by 12.5% and then rounded to the nearest value divisible by 4. 6 is added to compensate for a small increase in size when the array length is small.

With this formula, the allocated space for the array will grow up in these steps: 0, 4, 8, 16, 24, 32, 40, 52, 64 …

Now suppose we have a list with 24 elements. If we start removing elements from the list and when the list length will reach 11 elements (less than half of the allocated space for array), then the condition in the first mentioned block of code will fail and `new_allocated` will be calculated as 16.
