[![GoDoc](https://godoc.org/aqwari.net/xml?status.svg)](https://godoc.org/aqwari.net/xml) [![Build Status](https://travis-ci.org/droyo/go-xml.svg?branch=master)](https://travis-ci.org/droyo/go-xml)

## This fork

This is forked to get around
    https://github.com/droyo/go-xml/issues/32
    https://github.com/golang/go/issues/11939

That issue means when marshalling a struct into xml,
based upon code generated from an xsd containing a `<choice>` element,
then the resulting xml would contain every single option, even if empty.

Let's say an XSD has
```
    <complexType name="ChooseOne">
        <choice>
            <element name="foo" type="FooType"/>
            <element name="bar" type="BarType"/>
        </choice>
    </complexType>
```
You'd get a struct like
```
    type ChooseOne struct {
	Foo FooType `xml:"foo,omitempty"`
	Bar BarType `xml:"bar,omitempty"`
    }
```
And `xml.Marshal(&ChooseOne{})` generates  `"<ChooseOne><foo></foo><bar></bar></ChooseOne>"`

## Hat tip

This fork builds upon the branch `omitempty-structs` from the main repo https://github.com/droyo/go-xml

## More examples

Let's say an XSD has
```
    <complexType name="Foo">
        <choice>
            <element name="bar1" type="BarType" minOccurs="0"/>
            <element name="bar2" type="BarType"/>
        </choice>
    </complexType>
```


Then go-xml main would produce the struct `Foo` in this sample
```
package main

import (
    "encoding/xml"
    "fmt"
)

type Foo struct {
    Bar1 Bar  `xml:"bar1,omitempty"`
    Bar2 Bar  `xml:"bar2"`
}
// encoding.xml.Marshal generates "<Foo><bar1><Bip></Bip></bar1><bar2><Bip></Bip></bar2></Foo>" :(

type Bar struct {
    Bip string `xml:"Bip"`
}

func main() {
    buf, err := xml.Marshal(&Foo{})
    fmt.Printf("buf is (%s)\n", buf)
    fmt.Printf("err is (%+v)\n", err)
}
```


The `omitempty-structs` branch in main repo would generate this
```
type Foo struct {
    Bar1 *Bar  `xml:"bar1,omitempty"`
    Bar2 *Bar  `xml:"bar2"`
}
// encoding.xml.Marshal generates "<Foo></Foo>"
// If the xsd schema indicated Bar1 had minOccurs=0 and Bar2 had minOccurs=1
// then xsdgen noted that and added the omitempty tag.
// But branch `omitempty-structs` lets us fully omit Bar2 even though it had minOccurs=1
```

The `omitempty-structs-2` branch in this fork would generate
```
type Foo struct {
    Bar1 *Bar  `xml:"bar1,omitempty"`
    Bar2  Bar  `xml:"bar2"`
}
// encoding.xml.Marshal generates "<Foo><bar2><Bip></Bip></bar2></Foo>"
// So any required field is generated, even if it is empty
```

The difference between branch `omitempty-structs` and `omitempty-structs-2` is just a few lines.

`omitempty-structs-2` was then merged to `main` (on this fork) to pick up various bug fixes.


## Installation

Requires go 1.9 or greater for golang.org/x/html dependency.

```
go get aqwari.net/xml/...
```

This repository contains a collection of Go packages for working
with XML, with the ultimate goal of enabling code generation based
on XML documents.

- The `xmltree` package converts xml documents to a tree data
  structure, and provides convenient methods for manipulating and
  searching through that tree.
- The `xsd` package implements a parser for XML Schema. It takes
  some liberties from the specification, and would need some work for
  use as a validator, but it handles type inheritance and XML namespaces
  in a relatively sane way.
- The `xsdgen` package provides a customizable code generator that
  generates Go type declarations and marshal/unmarshal methods for
  an XML Schema.
- The `wsdl` package parses Web Service Definition Language (WSDL)
  files, which describe a (usually) SOAP web service.
- The `wsdlgen` package generates Go source code from WSDL files.
- The `xsdgen` and `wsdlgen` commands generate Go code with default
  settings and are suitable for use with `go generate`.

The directory wsdlgen/examples contains packages that were (mostly)
automatically generated using the wsdlgen package. You can run

	go generate

within the subdirectories to re-generate the code if you make changes
to the wsdlgen package.
This code is still very rough around the edges, but I have succesfully
used it to generate type declarations for some pretty complex XML
schema from an Apache Axis application. There are github issues
opened for missing functionality.
