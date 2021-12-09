+++
title = "grpctl"
outputs = ["Reveal"]
+++

# grpctl
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
grpcurl accounts-preprod.example.com:443 AccountsAPI ListAccounts -H 'Authorization: Basic Foobar' -d '{"accountfield1": "foo"}'

grpcurl petservice-preprod.example.com:443 PetService ListPets  -H 'Authorization: Basic Foobar' -d '{"petservicefield2": "bar"}'

grpcurl transactions-preprod.example.com:443 TransactionAPI ListTransctions -H 'Authorization: Basic Foobar' -d '{"transactionsfield2": "blah"}'
```

- Difficult to write, need to remember Service, method, host, fields

---

#### cobra





---
