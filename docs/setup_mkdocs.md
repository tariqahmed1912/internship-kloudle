## Setting up MkDocs

Install mkdocs and theme 'material'

Create a private git repository.

Create a Personal Access Token (Settings -> Developer Settings -> Personal Access Tokens).

Clone repository.

```bash
git clone https://<ACCESS_TOKEN>@github.com/tariqahmed1912/cloud-native
```

Go into the newly created directory and run the following commands:

```bash
cd cloud-native
mkdocs new .
mkdocs build --clean
```

Make changes in the .yml file and add Markdown files.

```bash
git add . && git commit -m "Commit changes" && git push -u origin main
```

