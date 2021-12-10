+++
title = "grpctl"
outputs = ["Reveal"]
+++

# grpctl

Automatically generate a cli for your apis

- @joshjcarp
  
- [github.com/joshcarp/grpctl](https://github.com/joshcarp/grpctl)
- [gophers slack #grpctl](https://gophers.slack.com/archives/C02CAH9NP7H)
- [joshcarp.com/talks-grpctl](https://joshcarp.com/talks-grpctl)
- [joshcarp.com](https://joshcarp.com)


---

Task: interact with a google grpc api: 
- [billingctl proto](https://github.com/googleapis/googleapis/blob/master/google/cloud/billing/v1/cloud_billing.proto#L34)
- [generated go code](https://github.com/googleapis/go-genproto/blob/main/googleapis/cloud/billing/v1/cloud_catalog.pb.go)


```bash

auth=$(gcloud auth application-default print-access-token)
billingctl -H="Authorization: Bearer $auth" CloudCatalog ListServices

```

---

## Problem

```bash
.
├── BUILD.bazel
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── Makefile
├── PACKAGES.md
├── README.md
├── SECURITY.md
├── WORKSPACE
├── api-index-v1.json
├── gapic
│   └── metadata
│       ├── BUILD.bazel
│       └── gapic_metadata.proto
├── google
│   ├── BUILD.bazel
│   ├── actions
│   │   ├── sdk
│   │   │   └── v2
│   │   │       ├── BUILD.bazel
│   │   │       ├── account_linking.proto
│   │   │       ├── account_linking_secret.proto
│   │   │       ├── action.proto
│   │   │       ├── actions_grpc_service_config.json
│   │   │       ├── actions_sdk.proto
│   │   │       ├── actions_testing.proto
│   │   │       ├── actions_v2.yaml
│   │   │       ├── config_file.proto
│   │   │       ├── conversation
│   │   │       │   ├── BUILD.bazel
│   │   │       │   ├── intent.proto
│   │   │       │   ├── prompt
│   │   │       │   │   ├── BUILD.bazel
│   │   │       │   │   ├── content
│   │   │       │   │   │   ├── BUILD.bazel
│   │   │       │   │   │   ├── canvas.proto
│   │   │       │   │   │   ├── card.proto
│   │   │       │   │   │   ├── collection.proto
│   │   │       │   │   │   ├── content.proto
│   │   │       │   │   │   ├── image.proto
│   │   │       │   │   │   ├── link.proto
│   │   │       │   │   │   ├── list.proto
│   │   │       │   │   │   ├── media.proto
│   │   │       │   │   │   └── table.proto
│   │   │       │   │   ├── prompt.proto
│   │   │       │   │   ├── simple.proto
│   │   │       │   │   └── suggestion.proto
│   │   │       │   └── scene.proto
│   │   │       ├── data_file.proto
│   │   │       ├── event_logs.proto
│   │   │       ├── files.proto
│   │   │       ├── interactionmodel
│   │   │       │   ├── BUILD.bazel
│   │   │       │   ├── conditional_event.proto
│   │   │       │   ├── entity_set.proto
│   │   │       │   ├── event_handler.proto
│   │   │       │   ├── global_intent_event.proto
│   │   │       │   ├── intent.proto
│   │   │       │   ├── intent_event.proto
│   │   │       │   ├── prompt
│   │   │       │   │   ├── BUILD.bazel
│   │   │       │   │   ├── content
│   │   │       │   │   │   ├── BUILD.bazel
│   │   │       │   │   │   ├── static_canvas_prompt.proto
│   │   │       │   │   │   ├── static_card_prompt.proto
│   │   │       │   │   │   ├── static_collection_browse_prompt.proto
│   │   │       │   │   │   ├── static_collection_prompt.proto
│   │   │       │   │   │   ├── static_content_prompt.proto
│   │   │       │   │   │   ├── static_image_prompt.proto
│   │   │       │   │   │   ├── static_link_prompt.proto
│   │   │       │   │   │   ├── static_list_prompt.proto
│   │   │       │   │   │   ├── static_media_prompt.proto
│   │   │       │   │   │   └── static_table_prompt.proto
│   │   │       │   │   ├── static_prompt.proto
│   │   │       │   │   ├── static_simple_prompt.proto
│   │   │       │   │   ├── suggestion.proto
│   │   │       │   │   └── surface_capabilities.proto
│   │   │       │   ├── scene.proto
│   │   │       │   ├── slot.proto
│   │   │       │   └── type
│   │   │       │       ├── BUILD.bazel
│   │   │       │       ├── class_reference.proto
│   │   │       │       ├── entity_display.proto
│   │   │       │       ├── free_text_type.proto
│   │   │       │       ├── regular_expression_type.proto
│   │   │       │       ├── synonym_type.proto
│   │   │       │       └── type.proto
│   │   │       ├── localized_settings.proto
│   │   │       ├── manifest.proto
│   │   │       ├── release_channel.proto
│   │   │       ├── settings.proto
│   │   │       ├── surface.proto
│   │   │       ├── theme_customization.proto
│   │   │       ├── validation_results.proto
│   │   │       ├── version.proto
│   │   │       └── webhook.proto
│   │   └── type
│   │       ├── BUILD.bazel
│   │       ├── date_range.proto

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

- Easier to interact ✅
- Difficult to maintain ❌
- Every team will have varying implementations ❌

--- 

### grpctl

```go
package main

import (
	"context"
	"log"
	"os"

	"github.com/joshcarp/grpctl"
	"github.com/spf13/cobra"
	"google.golang.org/genproto/googleapis/cloud/billing/v1"
)

// Example call:
// billingctl -H="Authorization: Bearer $(gcloud auth application-default print-access-token)" CloudBilling ListBillingAccounts.
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
	if err != nil {
		log.Print(err)
	}
	if err := grpctl.RunCommand(cmd, context.Background()); err != nil {
		log.Print(err)
	}
}
```

---

### grpctl

- [protoreflect.FileDescriptor](https://github.com/googleapis/go-genproto/blob/3a66f561d7aa4010d9715ecf4c19b19e81e19f3c/googleapis/cloud/billing/v1/cloud_catalog.pb.go#L1059)

{{< figure src="filedescriptor.png" width="70%" >}}

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

#### Bonus

![](https://raw.githubusercontent.com/joshcarp/grpctl/main/grpctl.svg)

--- 

### Links

- [github.com/joshcarp/grpctl](https://github.com/joshcarp/grpctl)
- [gophers slack #grpctl](https://gophers.slack.com/archives/C02CAH9NP7H)
- [joshcarp.com/talks-grpctl](https://joshcarp.com/talks-grpctl)
- [joshcarp.com](https://joshcarp.com)