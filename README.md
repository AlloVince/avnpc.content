# Content storage of avnpc.com

Avnpc posts content remote storage, compatible with hexo

```
npm install -g hexo-cli
hexo init avnpc.content
cd avnpc.content
```
Write blog posts as Markdown files under `source/_posts`.

This repository is the content source, not a canonical long-running local service. The current product preview is owned by `avnpc.front`, which reads this repository directly at build/dev time. `hexo server` may still be used as an explicit legacy/manual content diagnostic, but it is not the normal system development entry point and is not managed as a repository lifecycle service.

Create a new post:

```
hexo new post "title" -s "slug"
```

Start hexo server

```
hexo server
```
