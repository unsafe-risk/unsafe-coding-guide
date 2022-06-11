# golang struct parse with go/parser

```go
package main

import (
	"fmt"
	"go/ast"
	"go/parser"
	"go/token"
	"strings"
)

func main() {
	src := `package foo

type A struct {
	B int
	C struct {
		D int
		E string
		F struct {
			G *int
		}
	}
}

type Z struct {
	Y [10]float64
	T []int
	X *struct {
		D int
		E string
		F struct {
			G *int
		}
	}
}

type G[T any] struct {
	X T
	Y *T
}

type U[R any] struct {
	L G[int]
	P R
	Q *R
}
`

	rs, err := SearchStructs(src)
	if err != nil {
		panic(err)
	}
	for k, v := range rs {
		fmt.Printf("%s:\n", k)
		for _, r := range v {
			fmt.Printf("\t%s\n", r)
		}
	}
}

type Field struct {
	Name string
	Type string
}

func explore(parents []string, fields []*ast.Field) ([]Field, error) {
	results := []Field{}
	for _, field := range fields {
		if v, ok := field.Type.(*ast.Ident); ok {
			results = append(results, Field{
				Name: strings.Join(append(parents, field.Names[0].Name), "."),
				Type: v.Name,
			})
			continue
		}
		if v, ok := field.Type.(*ast.ArrayType); ok {
			length := ""
			if v.Len != nil {
				length = v.Len.(*ast.BasicLit).Value
			}
			results = append(results, Field{
				Name: strings.Join(append(parents, field.Names[0].Name), "."),
				Type: "[" + length + "]" + v.Elt.(*ast.Ident).Name,
			})
			continue
		}
		if v, ok := field.Type.(*ast.StarExpr); ok {
			if v, ok := v.X.(*ast.Ident); ok {
				results = append(results, Field{
					Name: strings.Join(append(parents, field.Names[0].Name), "."),
					Type: "*" + v.Name,
				})
				continue
			}
			if s, ok := v.X.(*ast.StructType); ok {
				rs, err := explore(append(parents, "*"+field.Names[0].Name), s.Fields.List)
				if err != nil {
					return nil, err
				}
				results = append(results, rs...)
				continue
			}
		}
		if v, ok := field.Type.(*ast.StructType); ok {
			rs, err := explore(append(parents, field.Names[0].Name), v.Fields.List)
			if err != nil {
				return nil, err
			}
			results = append(results, rs...)
			continue
		}
		if v, ok := field.Type.(*ast.IndexExpr); ok {
			results = append(results, Field{
				Name: strings.Join(append(parents, field.Names[0].Name), "."),
				Type: "[" + v.Index.(*ast.Ident).Name + "]" + v.X.(*ast.Ident).Name,
			})
			continue
		}
		fmt.Printf("%T\n", field.Type)
	}
	return results, nil
}

func SearchStructs(code string) (map[string][]Field, error) {
	fset := token.NewFileSet()

	f, err := parser.ParseFile(fset, "", code, parser.Mode(0))
	if err != nil {
		return nil, fmt.Errorf("SearchStructs: parser.ParseFile: %v", err)
	}

	m := map[string][]Field{}

	for _, decl := range f.Decls {
		genDecl, ok := decl.(*ast.GenDecl)
		if !ok {
			continue
		}
		for _, spec := range genDecl.Specs {
			typeSpec, ok := spec.(*ast.TypeSpec)
			if !ok {
				continue
			}
			structType, ok := typeSpec.Type.(*ast.StructType)
			if !ok {
				continue
			}
			rs, err := explore([]string{typeSpec.Name.Name}, structType.Fields.List)
			if err != nil {
				return nil, fmt.Errorf("SearchStructs: explore: %v", err)
			}
			m[typeSpec.Name.Name] = rs
		}
	}
	return m, nil
}
```

```bash
➜  go go run .
A:
        {A.B int}
        {A.C.D int}
        {A.C.E string}
        {A.C.F.G *int}
Z:
        {Z.Y [10]float64}
        {Z.T []int}
        {Z.*X.D int}
        {Z.*X.E string}
        {Z.*X.F.G *int}
G:
        {G.X T}
        {G.Y *T}
U:
        {U.L [int]G}
        {U.P R}
        {U.Q *R}
```

#go #parser #struct