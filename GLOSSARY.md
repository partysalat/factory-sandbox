# Blog

A small single-author blog. Readers browse published posts. The author writes and publishes posts.

## Language

**Post**:
A written entry with a title and a markdown body, identified by its slug.
_Avoid_: Article, entry, page

**Slug**:
The unique, immutable, URL-safe name of a post. It is the post's identity.
_Avoid_: ID, permalink, handle

**Draft**:
A post that only the author can see.
_Avoid_: Unpublished post, WIP

**Published**:
The state of a post that every reader can see. A post has a published date, set the first time it is published.
_Avoid_: Live, public

**Unpublish**:
Return a published post to draft. The post keeps its published date.
_Avoid_: Archive, hide

**Author**:
The one person who writes, publishes, and deletes posts. Proves identity with the admin token.
_Avoid_: Admin, owner, editor, user

**Reader**:
An anonymous visitor who sees published posts only.
_Avoid_: Visitor, user, subscriber

**Admin token**:
The single secret the author sends to prove identity for writes.
_Avoid_: API key, password, session
