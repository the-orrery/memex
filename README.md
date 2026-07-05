# memex

`memex` 是一个 local-first 的 Markdown 知识库检索引擎。它把多个源仓里的
Markdown note 编译成可重建索引,对外提供 lexical(BM25)、semantic(向量)与
hybrid(weighted RRF) 检索。

数据默认留在本机:源仓是普通 Git 仓里的 Markdown + frontmatter,编译产物在
用户本地数据目录,向量库和 embedding endpoint 都通过环境变量配置。只用 lexical
lane 时不需要外部服务。

## 入口

| 命令 | 用途 |
|---|---|
| `memex` | 读路径:检索、召回、本地用量统计。 |
| `memex-sync` | 写路径:扫描源仓、编译、同步向量索引。 |

## 源仓配置

memex 通过 `kb-sources.toml` 声明需要索引的源仓:

```toml
source_root = "~/projects"

[[source]]
name = "docs"

[[source]]
name = "notes"
path = "~/projects/notes"
```

解析顺序:

1. `$KB_SOURCES` 指定的 TOML 文件。
2. 否则 `$KB_SOURCE_ROOT` 下的 `kb-sources.toml`。
3. 都不可用时降级到内置示例源,并打印 warning。

## 外部服务

semantic lane 依赖一个 OpenAI-compatible embedding endpoint 和一个 qdrant
实例。常用配置:

- `KB_SEARCH_QDRANT_URL`:qdrant URL,默认 `http://127.0.0.1:6333`。
- `KB_SEARCH_EMBEDDING_URL`:embedding endpoint,默认 `http://127.0.0.1:3002/v1/embeddings`。
- `KB_SEARCH_EMBEDDING_MODEL`:embedding model name。
- `KB_SEARCH_EMBEDDING_DIMENSIONS`:embedding vector dimension。
- `MEMEX_BEARER`:可选 bearer credential。
- `MEMEX_CA_BUNDLE`:可选 CA bundle 路径。

## 安装和开发

```sh
uv tool install --force .
uv run memex --help
uv run memex-sync --help
uv run pytest
```

常用命令:

```sh
memex recall "查询文本"
memex recall "查询文本" --format json
memex query "查询文本" --lane lexical
memex-sync compile
memex-sync sync --apply
```

写路径默认 dry-run,只有显式传 `--apply` 才会写向量库或落盘 compiled 产物。

## 文档

架构和模块地图见 [`docs/architecture.md`](docs/architecture.md)。

## 许可证

MIT
