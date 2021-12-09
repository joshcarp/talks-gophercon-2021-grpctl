+++
title = "grpctl"
outputs = ["Reveal"]
+++

# grpctl

Automatically generate a cli tool for your apis

---

## Problem

```bash
├── accounts
│         └── accounts.proto
├── cards
│         └── cards.proto
├── foobar
│         └── foobar.proto
├── petservice
│         └── petservice.proto
└── transactions
    └── transactions.proto
... a lot more
```

---
### Need a way to interact with gRPC APIs

#### grpcurl

```bash
grpcurl accounts-preprod.example.com:443 AccountsAPI ListAccounts \
        -H 'Authorization: Basic Foobar' -d '{"accountfield1": "foo"}'

grpcurl petservice-preprod.example.com:443 PetService ListPets \
        -H 'Authorization: Basic Foobar' -d '{"petservicefield2": "bar"}'

grpcurl transactions-preprod.example.com:443 TransactionAPI ListTransctions \ 
        -H 'Authorization: Basic Foobar' -d '{"transactionsfield2": "blah"}'
```

- Difficult to write ❌
- A lot to remember ❌

---

#### cobra

```bash
fooctl service1 ListFoo <args>
fooctl service2 ListBar --field1=foo
fooctl service2 ListBar --data='{"field2": "blah"}'
```

- Easier to write ✅
- Difficult to maintain ❌
- Every team will have varying implementations ❌

--- 

### grpctl

{{< figure src="filedescriptor.png" width="70%" >}}

---

#### What is protoreflect.FileDescriptor??

- Has information about Services, Methods, Messages, etc

- Same information that gRPC reflection does

```go

type FileDescriptor interface {

	Descriptor // Descriptor.FullName is identical to Package

	// Path returns the file name, relative to the source tree root.

	Path() string // e.g., "path/to/file.proto"

	// Package returns the protobuf package namespace.

	Package() FullName // e.g., "google.protobuf"

	// Imports is a list of imported proto files.

	Imports() FileImports

	// Enums is a list of the top-level enum declarations.

	Enums() EnumDescriptors

	// Messages is a list of the top-level message declarations.

	Messages() MessageDescriptors

	// Extensions is a list of the top-level extension declarations.

	Extensions() ExtensionDescriptors

	// Services is a list of the top-level service declarations.

	Services() ServiceDescriptors

	// SourceLocations is a list of source locations.

	SourceLocations() SourceLocations

	// contains filtered or unexported methods

}

```

---
#### grpctl
| protoreflect | cobra | example
| -- | -- | -- |
| ServiceDescriptor| Command | `fooctl PetService`
| MethodDescriptor | Command | `fooctl PetService ListPets`
| MessageDescriptor| flags | `fooctl PetService ListPets --name=brian`
    
---

#### example

![](https://raw.githubusercontent.com/joshcarp/grpctl/main/examplectl.gif)

---

### Installation

```go
func main() {
	cmd := &cobra.Command{
		Use:   "billingctl",
		Short: "an example cli tool for the gcp billing api",
	}
	err := grpctl.BuildCommand(cmd,
		grpctl.WithArgs(os.Args),
		grpctl.WithFileDescriptors(
			billing.File_google_cloud_billing_v1_cloud_billing_proto,
			billing.File_google_cloud_billing_v1_cloud_catalog_proto,
		),
	)
	cobra.CheckErr(err)
	cobra.CheckErr(cmd.Execute())
}
```
---

#### Bonus

![](https://raw.githubusercontent.com/joshcarp/grpctl/main/grpctl.svg)