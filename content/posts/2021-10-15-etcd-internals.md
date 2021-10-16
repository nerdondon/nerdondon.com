---
title: "[Notes]: etcd Internals"
date: 2021-10-15T00:00:00-08:00
description: "Notes on etcd internals"
tags: [notes, database, distributed, etcd, raft]
logoText: "etcdctl get /nerdondon/notes/2021-10-15-etcd-internals"
---

## Prelude

For now, mostly just some cool articles that were very helpful in understanding etcd internals.

## Notes

- etcd first sends transaction requests to it's Raft engine. This effectively serves as a
  write-ahead log. Once consensus on the transaction is reached, the transaction is committed to
  etcd's underlying key-vale store.

- etcd uses [bbolt](https://github.com/etcd-io/bbolt) as its durable store, which is based on a B+
  tree variation

- etcd will persist Raft cluster information/membership
  [in](https://github.com/etcd-io/etcd/blob/c0ab5708a560c8e7e2217169ca94cccde9b28982/server/etcdserver/api/membership/cluster.go#L386-L405)
  [it's](https://github.com/etcd-io/etcd/blob/main/server/storage/schema/membership.go#L47-L58)
  storage backend
  - Old pointer to older code from the
    [mailing list](https://groups.google.com/g/etcd-dev/c/LRu0XILQZIw)

## Useful references

- [Michael Gasch - Onwards to the Core: etcd](https://www.mgasch.com/2021/01/listwatch-part-1/)
- [Pierre Zemb - Notes about ETCD](https://pierrezemb.fr/posts/notes-about-etcd/)
- [Dmitry Dolgov - Evolution of tree data structures for indexing: more exciting than it sounds](https://erthalion.info/2020/11/28/evolution-of-btree-index-am/)
- [Michelle Nguyen - How etcd works and 6 tips to keep in mind](https://blog.px.dev/etcd-6-tips/)
