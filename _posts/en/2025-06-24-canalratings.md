---
title: "Canal+ Ratings : Firefox extension"
description: "Adds IMDb, Rotten Tomatoes and Allocine ratings on Canal+ streaming platform"
date: 2025-06-24
categories: [Website]
tags: [website, firefox, extension, javascript]
media_subpath: /assets/img/posts/canalratings
lang: en
image: 
    path: .png
---

## Ratings sources
- IMDb, Rotten Tomatoes, Allocine (FR)

### API
- Hard to get and expensive

### Scraping
- Search engine

### Prevent "Too Many Requests" error and bot detection
- Implement cache and queue system to add delay between each requests

## Customize Canal+ page
- Show rating on details page of each movie
- Show rating on thumbnails of all movies
- Make detail page rating first in the request queue
- Show loading spinner to see if rating is waiting to be fetched

## Bundle a Firefox extension

### Manifest

### Option page

### Publish on Firefox Store

### Migrate to Manifest V3 and Chrome

## Future improvements
- Improve rating search
- Add more rating sources
- Support other streaming platforms