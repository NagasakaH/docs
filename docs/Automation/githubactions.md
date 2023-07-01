# GitHub Actions覚書

.github/workflows配下にymlを生成することで任意のジョブを有効化することができる

## Dockerイメージを自動ビルドしてDocker HubにPushする

リポジトリのsecretsのdockerhub_usernameとdockerhub_tokenの設定を追加する必要あり

```yml
name: docker_build

on:
  push:
    branches:
      - "main"
    paths:
      - "<イメージに影響のあるパスを記載する>

jobs:
  build_mkdocs_image:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: "{{defaultContext}}:<Dockerfileのあるパス>"
          push: true
          tags: <生成するイメージ名>
```

## GitHub PagesにDockerイメージを使ってビルド後に自動デプロイする

```yml
on:
  push:
    branches:
      - "main"
    path:
      - "<ページの内容に影響のあるパスを記載する>"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v2
      - name: Run the build process with Docker
        uses: addnab/docker-run-action@v3
        with:
          image: <ページの生成に必要なイメージ名>
          options: -v ${{ github.workspace }}:/work
          run: |
            <ページを生成するためのコマンド類>
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: '<生成したページのパス>'

  deploy:
    # Add a dependency to the build job
    needs: build

    # Grant GITHUB_TOKEN the permissions required to make a Pages deployment
    permissions:
      pages: write # to deploy to Pages
      id-token: write # to verify the deployment originates from an appropriate source

    # Deploy to the github-pages environment
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    # Specify runner + deployment step
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2 # or the latest "vX.X.X" version tag for this action
        with:
          path: '<生成したページのパス>'
```
