<!-- omit in toc -->
# 環境構築

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

## 1. プロジェクトの作成

```bash
# プロジェクトの作成
$ yarn global add @nestjs/cli
$ nest new project-name
```

## 2. Formatter + Lint設定

### 2.1. VSCodeの前提プラグイン

```json title=".vscode/extensions.json"
{
 "recommendations": [
  "dbaeumer.vscode-eslint",
  "editorconfig.editorconfig",
  "oderwat.indent-rainbow",
  "esbenp.prettier-vscode"
 ]
}
```

### 2.2. googleのスタイルガイドを追加

```bash
# gtsをインストールしてセットアップする
# tsファイル等をoverwriteすると正常に動かない可能性があるので対話は要注意
$ yarn gts init --yarn # Overwriteは基本的にNo lintはgts lintに置き換えても良い
$ rm prettirrc # 実行後にprettierrcとprettirrc.jsが混在するのでprettierrcを削除する
$ yarn gts fix # backet white space等が引っかかるので修正しておく
```

### 2.3. VSCode用設定

```json title=".vscode/settings.json"
{
  "editor.renderWhitespace": "all",
  "editor.formatOnSave": true,
  "prettier.configPath": ".prettierrc.js",
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

## 3. デバッグ用の設定を追加する

### 3.1. 設定ファイルの作成

```json title=".vscode/launch.json"
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "pwa-node",
      "name": "Launch via yarn",
      "request": "launch",
      "runtimeArgs": [
          "run",
          "start:debug"
      ],
      "runtimeExecutable": "yarn",
      "skipFiles": [
          "<node_internals>/**"
      ],
      "sourceMaps": true,
      "envFile": "${workspaceFolder}/.env",
      "cwd": "${workspaceRoot}",
      "console": "integratedTerminal",
      "protocol": "inspector"
    }
  ]
}
```

## 4. 設定ファイルの追加

### 4.1. .envファイルの作成

.gitignoreに.envを追加  
.envファイルを作成する  
(元の項目等は.env.template等に書いておいてそこから複製するようにすると良い)

```bash title=".env"
# 必要に応じて書き換える
COMPOSE_PROJECT_NAME=nestjsTemplate
NESTJS_PORT=8080
NESTJS_PRODUCTION_PORT=8080
POSTGRES_CONTAINER_NAME=postgres
POSTGRES_PORT=5432
POSTGRES_PASSWORD=password
POSTGRES_USER=user
POSTGRES_DATABASE=database
PGDATA=/var/lib/postgresql/data/pgdata
```

### 4.2. パッケージのインストール

```bash
# envファイルを扱うためのパッケージを追加
$ yarn add @nestjs/config
```

### 4.3. configurationの作成

```typescript title="src/config/configuration.ts"
export default () => ({
  port: parseInt(process.env.NESTJS_PORT, 10) || 3000,
  database: {
    host: process.env.POSTGRES_HOST,
    password: process.env.POSTGRES_PASSWORD,
    user: process.env.POSTGRES_USER,
    database: process.env.POSTGRES_DATABASE
    port: parseInt(process.env.POSTGRES_PORT, 10) || 5432,
  },
});
```

### 4.4. configurationを読み込むように変更

```diff title="src/app.module.ts"
 import {Module} from '@nestjs/common';
 import {AppController} from './app.controller';
 import {AppService} from './app.service';
+import {ConfigModule} from '@nestjs/config';
+import configuration from './config/configuration';
 
 @Module({
-  imports: [],
+  imports: [ConfigModule.forRoot({load: [configuration]})],
   controllers: [AppController],
   providers: [AppService],
 })
```

## 5. TypeORMの追加

### 5.1. ライブラリの追加

```bash
# データベースの接続に必要なパッケージをインストール
$ yarn add typeorm pg @nestjs/typeorm
```

### 5.2. データベース接続情報の追加

```typescript title="src/database.providers.ts"
import {ConfigService} from '@nestjs/config';
import {DataSource} from 'typeorm';

export const databaseProviders = [
  {
    provide: 'DATA_SOURCE',
    inject: [ConfigService],
    useFactory: async (configService: ConfigService) => {
      const dataSource = new DataSource({
        type: 'postgres',
        host: configService.get<string>('database.host'),
        port: configService.get<number>('database.port'),
        username: configService.get<string>('database.user'),
        password: configService.get<string>('database.password'),
        database: configService.get<string>('database.database'),
        entities: [__dirname + '/../**/*.entity{.ts,.js}'],
        synchronize: false, // 勝手にEntityとDBのカラムを同期してしまうので絶対にtrueにしない
      });

      return dataSource.initialize();
    },
  },
];
```

```typescript title="src/database.module.ts"
import {Module} from '@nestjs/common';
import {databaseProviders} from './database.providers';

@Module({
  providers: [...databaseProviders],
  exports: [...databaseProviders],
})
export class DatabaseModule {}
```

### entityの追加

仮でentityを作成しておく(動作確認用)

```typescript title="src/entities/user.ts"
import {Entity, Column, PrimaryGeneratedColumn} from 'typeorm';

@Entity("users")
export class User {
  @PrimaryGeneratedColumn()
  @Generated('uuid')
  id: number;

  @Column({length: 255})
  name: string;

  @Column({length: 255})
  email: string;

  @Column('int')
  lastLoggedIn: number; // タイムスタンプ(UTC)
}
```

### 5.3. マイグレーション設定

## 6. 開発/本番用のDockerファイルの配置

開発用(DBのみ)

```yaml title="docker/development/compose.yml"
volumes:
  pg_data:


services:
  postgres:
    image: postgres:14 # debian slimeベースのPostgres14の最新バージョン
    env_file: ../../.env
    volumes:
      - pg_data:/var/lib/postgresql/data
    ports:
      - $POSTGRES_PORT:5432

```

本番用(DB+NestJS本体)

```yaml title="docker/production/compose.yml"
volumes:
  pg_data:


services:
  nestjs:
    build:
      context: ../../
      dockerfile: ./docker/production/Dockerfile
    ports:
      - ${NESTJS_PRODUCTION_PORT}:${NESTJS_PORT}
    restart: on-failure

  postgres:
    image: postgres:14 # debian slimeベースのPostgres14の最新バージョン
    env_file: ../../.env
    volumes:
      - pg_data:/var/lib/postgresql/data
    restart: on-failure
```

```dockerfile title="docker/production/Dockerfile"
# Base image
FROM node:18

# Create app directory
WORKDIR /usr/src/app

# A wildcard is used to ensure both package.json AND package-lock.json are copied
COPY package*.json ./

# Install app dependencies
RUN yarn install

# Bundle app source
COPY . .

# Creates a "dist" folder with the production build
RUN yarn run build

# Start the server using the production build
CMD [ "node", "dist/main.js" ]
```
