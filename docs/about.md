> 2020-04-04

```
pip3 install --trusted-host pypi.douban.com -i http://pypi.douban.com/simple/  mkdocs mkdocs-material mkdocs-minify-plugin 
```

# 标题

## 标题一

Material for MkDocs provides the following template blocks:

| Block name   | Wrapped contents                                |
| ------------ | ----------------------------------------------- |
| `analytics`  | Wraps the Google Analytics integration          |
| `announce`   | Wraps the Announcement bar                      |
| `config`     | Wraps the JavaScript application config         |
| `content`    | Wraps the main content                          |
| `disqus`     | Wraps the disqus integration                    |
| `extrahead`  | Empty block to define additional meta tags      |
| `fonts`      | Wraps the webfont definitions                   |
| `footer`     | Wraps the footer with navigation and copyright  |
| `header`     | Wraps the fixed header bar                      |
| `hero`       | Wraps the hero teaser (if available)            |
| `htmltitle`  | Wraps the `<title>` tag                         |
| `libs`       | Wraps the JavaScript libraries (header)         |
| `scripts`    | Wraps the JavaScript application (footer)       |
| `source`     | Wraps the linked source files                   |
| `site_meta`  | Wraps the meta tags in the document head        |
| `site_nav`   | Wraps the site navigation and table of contents |
| `styles`     | Wraps the stylesheets (also extra sources)      |
| `tabs`       | Wraps the tabs navigation (if available)        |



## 标题二


``` sh
.
├─ assets/
│  ├─ images/                          # Images and icons
│  ├─ javascripts/                     # JavaScript
│  └─ stylesheets/                     # Stylesheets
├─ partials/
│  ├─ integrations/                    # 3rd-party integrations
│  ├─ language/                        # Localized languages
│  ├─ footer.html                      # Footer bar
│  ├─ header.html                      # Header bar
│  ├─ hero.html                        # Hero teaser
│  ├─ language.html                    # Localized labels
│  ├─ nav-item.html                    # Main navigation item
│  ├─ nav.html                         # Main navigation
│  ├─ search.html                      # Search box
│  ├─ social.html                      # Social links
│  ├─ source-date.html                 # Last updated date
│  ├─ source-link.html                 # Link to source file
│  ├─ source.html                      # Repository information
│  ├─ tabs-item.html                   # Tabs navigation item
│  ├─ tabs.html                        # Tabs navigation
│  ├─ toc-item.html                    # Table of contents item
│  └─ toc.html                         # Table of contents
├─ 404.html                            # 404 error page
├─ base.html                           # Base template
└─ main.html                           # Default page
```