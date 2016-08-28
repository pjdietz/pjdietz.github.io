# pjdietz.com

Jekyll site for pjdietz.com

## Local Development

Start Jekyll with:

```bash
docker-compose up -d
```

This does a few things:
- Builds the site
- Begins serving the site at http://localhost:4000
- Watches the local files for change and updates automatically

To update the site after making changes the watcher can't handle:

```bash
docker-compose restart
```
