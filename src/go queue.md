# queue in go

## single linked list

```go
package queue

import "sync"

type Element[T any] struct {
	value T
	next  *Element[T]
}

type Queue[T any] struct {
	head *Element[T]
	tail *Element[T]
	size int

	pool sync.Pool
}

func New[T any]() *Queue[T] {
	return &Queue[T]{
		nil,
		nil,
		0,
		sync.Pool{
			New: func() interface{} {
				return &Element[T]{}
			},
		},
	}
}

func (q *Queue[T]) Push(value T) {
	e := q.pool.Get().(*Element[T])
	e.value = value
	e.next = nil
	if q.tail == nil {
		q.head = e
	} else {
		q.tail.next = e
	}
	q.tail = e
}

func (q *Queue[T]) Pop() (value T, check bool) {
	if q.size == 0 {
		return value, false
	}
	e := q.head
	value = e.value
	q.pool.Put(e)
	q.head = q.head.next
	q.size--
	return value, true
}
```

## slice

```go
package queue

type Queue[T any] struct {
	list []T
}

func New[T any]() *Queue[T] {
	return &Queue[T]{}
}

func (q *Queue[T]) Push(value T) {
	q.list = append(q.list, value)
}

func (q *Queue[T]) Pop() (value T, check bool) {
	if len(q.list) == 0 {
		return value, false
	}
	value = q.list[0]
	q.list = q.list[1:]
	return value, true
}
```