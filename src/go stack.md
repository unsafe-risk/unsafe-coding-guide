# stack in go

## single linked list

```go
package stack

import "sync"

type Element[T any] struct {
	value T
	next  *Element[T]
}

type Stack[T any] struct {
	top  *Element[T]
	size int

	pool sync.Pool
}

func New[T any]() Stack[T] {
	return Stack[T]{
		nil,
		0,
		sync.Pool{
			New: func() interface{} {
				return &Element[T]{}
			},
		},
	}
}

func (s *Stack[T]) Push(value T) {
	e := s.pool.Get().(*Element[T])
	e.value = value
	e.next = s.top
	s.top = e
	s.size++
}

func (s *Stack[T]) Pop() (value T, check bool) {
	if s.size == 0 {
		return value, false
	}
	e := s.top
	value = e.value
	s.pool.Put(e)
	s.top = s.top.next
	s.size--
	return value, true
}
```

## slice

```go
package stack

type Stack[T any] struct {
	list []T
}

func New[T any]() Stack[T] {
	return Stack[T]{list: make([]T, 0)}
}

func (s *Stack[T]) Push(value T) {
	s.list = append(s.list, value)
}

func (s *Stack[T]) Pop() (value T, check bool) {
	if len(s.list) == 0 {
		return
	}

	value = s.list[len(s.list)-1]
	s.list = s.list[:len(s.list)-1]
	return value, true
}
```

#stack #go #linked_list