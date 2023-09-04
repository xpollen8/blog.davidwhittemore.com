---
title: "The Manifestor"
date: 2023-07-17
---

# The Code

I will be covering the software at https://github.com/xpollen8/manifestor

# The problem

* You have access to an Internet-connected web server.
* That machine has interesting content scattered around (.mp3, .xls, etc).
* You would like to add context and share some of those files in a machine-readable format.

> **Manifestor** allows you to share collections of curated files *with added context* via a web server,
in a manner which does not involve duplicating the originals.
> 
> **Manifestor** was written explicitly to expose curated files from my local network, but the following approach will work for _any_ web server.

# The solution - *Manifestor*

> Manifestor, as its name suggests, is constructed around simple human-readable "manifest files"
which are created by the person wishing to curate some files to the Internet.
> 
> These manifest files contain "extended attributes" and are delivered from a particular URL
> 
> Manifest files are recursive in nature: they may reference folders, or even external URLs.

# Example

The basic skeleton of a `manifest.json` file is:

```
{
  "title": "...",
  "description": {},
  "contents": {},
  "history: {}
}
```

Here is the expanded form:

```
{
  "title": "<title for this collection of files>"
  "description": {
    "summary": "<short summary of this collection of files>",
    "details": {
      "body": "<full description of this collection of files>",
      "date": "<date collection published>",
      "source": "<collection attribution>",
    }
  },
  "contents": {
    "file1.mp3": {
      "title": "<short name of item>",
      "name": "(optional)<to override 'file1.mp3'>",
      "date": "<date published>",
      "summary": "<short description of this item>,
      "details": {
        "body": "<full description of this item (history, source, etc)>,
        "source": "<item attribution>",
        "date": "<date item created>"
      }
    },
    "file2.csv": {
      ... as above
    },
    "folder1": { "type": "folder" },
    "https://ANOTHER_BUH.com/file.xxx": {
      ... as above
    }
  },
  "history": {
    {
      "date": "<manifest last updated>",
      "summary": "<description of changes made>"
    },
    {
      ... as above
    }
  }
}
```

# Of note!

Normally, plain files on a hard drive are context-less - metatdata is limited.

One important goal of *manifestor* is to better describe those files in a trackable manner.

The `manifest.json` file allows you to publish (and track) file provenance via the `.summary` and `.details` attributes.

Think of this as the ability to add **extended attributes** to otherwise context-less files.

Manifest "contents" may be of three "types":
1. plain files = 'file.mp3'
2. folders - to expose curated sub-collections
3. URLs - to pull in external sources to this collection

# Filled-in example:

`https://BUH.com/manifest.json` (manifest.json can be anywhere - `/folder/manifest.json`, etc)

```
{
  "title": "Hey! Here are some files I'm sharing with the world",
  "description": {
    "summary": "Publicly available assets from BUH.com.",
    "details": {
      "body": "You could go into great length describing this collection of files.",
      "date": "2023-07-10",
      "source": "David Whittemore"
    }
  },
  "contents": {
    "file1.mp3": {
      "title": "Just an mp3",
      "date": "2023-07-20",
      "summary: "Brief summary of file contents",
      "details": {
        "body": "Full explaination of this file, how it was created, what it contains, why it is important",
        "date": "2004-05-12",
        "source": "David Whittemore"
      }
    },
    "folder1": { "type": "folder" },
    "https://adjective.com/audio/pos.mp3": {
      "title": "Parts of Speech Theme Song",
      "name": "20040512_POS_Theme.mp3",
      "date": "2023-07-20",
      "details": {
        "body": "Created w/a single drum loop sample on an Ensoniq EPS16+",
        "date": "2004-05-12",
        "source": "David Whittemore"
      }
    }
  }
}
```

**Note** that this `manifest.json` file contains `contents.folder1` which of `contents.folder1.type === 'folder'`.

This means that a subsequent fetch can be made to `https://BUH.com/folder1/manifest.json` which would
populate the `contents.folder1` slot with the contents of `folder1.manifest.json`.

# Curating files: tricks and tips

Suppose you have two different hard drives attached to your machine which is running your web server:

+ `/Volumes/Disk1`
+ `/Volumes/Disk2`

And you want to share these files:
+ `/Volumes/Disk1/artist/jazz_butcher/music/smith.mp3`
+ `/Volumes/Disk1/artist/jazz_butcher/music/sex.mp3`
+ `/Volumes/Disk1/artist/jazz_butcher/images/spat.jpg`
+ `/Volumes/Disk2/family/history/photographs/mom.jpg`
+ `/Volumes/Disk2/family/history/photographs/dad.jpg`
+ `/Volumes/Disk2/family/history/oral_history.mp3`

# **Symlinks are your friend**

> Let's create a `manifest.json` file to expose some files from those drives **without** copying!

+ "Collect" the files to publish into a named directory:
```
  mkdir jazzbutcher
  cd jazzbutcher
  ln -s /Volumes/Disk1/artist/jazz_butcher/music/smith.mp3 .
  ln -s /Volumes/Disk1/artist/jazz_butcher/music/sex.mp3 .
  ln -s /Volumes/Disk1/artist/jazz_butcher/images/pat.jpg .
```

+ Next, create a `jazzbutcher/manifest.json` file which has as _"contents"_, the files `smith.mp3`, `sex.mp3`, and `pat.jpg`

> `jazzbutcher/manifest.json`
```
  ...
  "contents": {
    "smith.mp3": {
      ... name, description, details, etc
    },
    "sex.mp3": {
      ... name, description, details, etc
    },
    "pat.jpg": {
      ... name, description, details, etc
    }
  },
  ...
```

+ Do similar for the `family folder contents:

> `family/manifest.json`
```
  ...
  "contents": {
    "oral_history.mp3": {
      ... name, description, details, etc
    },
    "mom.jpg": {
      ... name, description, details, etc
    },
    "dad.jpg": {
      ... name, description, details, etc
    }
  },
  ...
```

+ Finally, create a top level `manifest.json` file which points to those two collections:

> `manifest.json`
```
  ...
  "contents": {
    "jazzbutcher": { "type": "folder" },
    "family": { "type": "folder" },
  }
  ...
```

# Using the *manifestor* code

> The *manifestor* software will fetch the top-level `manifest.json` file, and then recursively "fill-in"
each of the "folder" items with the recursively-read `folder/manifest.json` files.

> **Neat!**

`package.json`:

```
  {
    ...
    "manifestor": "git+https://github.com/xpollen8/manifestor.git",
    ...
  }
```

# In your code

```js
  import fetchManifest, * as manifestor from 'manifestor';

  {
    ...
    // where is the web server?
    const root = process.env.MANIFEST_SERVER;
    // lets fetch the fully-populated manifest JSON
    const manifest = await fetchManifest({ root, recurse: true });
    ...
  }
```
