# IELTS Learning Lab

> I built the IELTS vocabulary desk I kept wishing existed. Then one stubborn product problem turned into a research question.
>
> 我先做了一个自己真正会用的 IELTS 词汇训练器，后来又从中发现了一个值得研究的问题。

[Live demo / 在线体验](https://yizihao8288-coder.github.io/ielts-learning-lab/demo/) · [Research status / 研究进度](https://yizihao8288-coder.github.io/ielts-learning-lab/results/) · [完整中文说明](README.zh-CN.md)

![Listening practice interface](site/assets/app-listening.png)

## Introduction / 项目简介

### 中文

这个项目来自我准备 IELTS 时遇到的实际问题：词表很多，但很难找到一个符合自己复习习惯的工具。我希望能在同一个地方听单词、练拼写、阅读例句、收藏内容，并反复练习容易出错的词，所以做了这个训练器。

它现在包含听力、填空、阅读、写作、收藏、错题复习、导入导出、本地保存和浏览器朗读。老师不需要注册账号或配置 API Key，打开在线版就可以直接体验。

后来我开始尝试自动生成释义和例句，也发现模型即使返回了格式正确的数据，内容仍然可能不适合学习。因此，我进一步加入了格式检查、内容检查和失败修复，并把它作为项目中继续探索的问题。

### English

While preparing for IELTS, I found plenty of word lists but no single place that matched the way I actually reviewed: listen to a word, type it, read it in context, save it, and come back to the mistakes later. I wanted those actions to belong to one learning loop instead of several disconnected tools, so I built one.

The app now has listening, dictation, reading and writing modes, plus favourites, mistake review, import/export, browser storage and speech. It works without an account or an API key.

Later I experimented with generated definitions and examples. That exposed a more interesting failure: a model can return flawless JSON and still produce a poor learning item. The word may change, the Chinese meaning may contain no Chinese, or the example may never use the phrase it is supposed to teach.

That is why this repository has two connected halves:

- a practice tool I can genuinely use;
- a small experiment about making generated vocabulary entries structurally valid **and** useful enough to review.

## Design choices / 设计取舍

- **Works without a key / 无密钥也能使用。** I kept the core practice features in the browser so a teacher or learner can open the demo immediately. / 我把核心练习功能留在浏览器中，让老师或学习者打开网页就能体验。
- **Format and content are checked separately / 分开检查格式和内容。** Correct JSON does not guarantee a correct definition or a useful example. / JSON 格式正确，不代表释义正确或例句适合学习。
- **Repair once, then fall back / 最多修复一次。** If the generated item still fails, the app keeps the local dictionary content instead of retrying indefinitely. / 如果生成内容仍未通过检查，应用会保留本地词典内容，而不是不断重试。

## Open it before reading more

The [public demo](https://yizihao8288-coder.github.io/ielts-learning-lab/demo/) runs entirely in the browser. Nothing needs to be installed, and the static version never calls a model or the Python server.

To run the local version on Windows, install Python 3.10 or newer and double-click `启动雅思训练器.bat`. On macOS or Linux:

```bash
python3 run.py
```

The app opens at `http://127.0.0.1:8765`. Core use has no third-party package dependency. Runtime files stay inside `.runtime/` or other ignored local files.

## 项目中的研究问题（通俗版）

### 我遇到了什么问题

我尝试让 AI 自动生成单词、中文释义、英文释义和例句。它经常能把四项内容填得很整齐，但“格式正确”不代表“内容能学”：单词可能被换掉，中文释义里可能没有中文，例句也可能根本没有用到目标词。

所以我想验证一件具体的事：**给 AI 更明确的填写格式，再检查它写的内容，发现问题后只让它改一次，能不能减少这些错误？**

### 我比较哪三种方法

| 方法 | 用通俗的话解释 | 代码中的名称 |
|---|---|---|
| 方法一 | 只告诉 AI“请按指定格式回答” | `baseline` |
| 方法二 | 用程序严格规定必须填写哪四项 | `schema` |
| 方法三 | 在方法二之后检查内容；不合格时指出问题，只允许修改一次 | `guarded` |

为了公平，三种方法使用同一个模型、同一批单词和相同的学习字段。区别只在于“要求有多明确”和“生成后有没有检查、改错”。

### 怎样判断一条内容能不能用

程序会检查：目标词有没有被换掉、中文释义是不是真的包含中文、例句有没有完整使用目标词、句子是否过短或过长，以及有没有占位文字、重复内容或 AI 的解释性废话。之后还会把部分结果隐藏方法名称，再由人判断释义是否正确、例句是否自然、是否适合教学。

### 目前做到哪一步

生成、检查、最多修改一次和失败后使用本地词典的程序已经完成，12 项自动测试已经通过。计划中的 **100 个词 × 3 种方法 × 3 次生成，共 900 条结果**以及人工匿名评分还没有执行完，因此目前不能宣称方法三一定更好，也不会提前放结果图。

[研究进度页](https://yizihao8288-coder.github.io/ielts-learning-lab/results/)只展示目前真正完成的内容；需要查看数字、字段和保存方式时，可以继续阅读[详细研究方案](docs/research-protocol.md)。研究设计参考了 [Wang 等（2023）](https://aclanthology.org/2023.nlp4dh-1.7/)、[JSONSchemaBench](https://arxiv.org/abs/2501.10868) 和 [Structured Output Benchmark](https://arxiv.org/abs/2604.25359)。

## One product, two ways to run it

```text
Public demo                         Optional local enhancement
Browser UI                          run.py (Python standard library)
├─ practice modes                   ├─ local save/load
├─ local dictionary                 ├─ POST /api/v1/generate-item
├─ browser storage                  └─ guarded generation pipeline
└─ browser speech                       ├─ strict schema
                                         ├─ semantic validation
                                         └─ one repair maximum
```

The public demo stays deliberately simple. The local server adds saving and optional online generation, but failure never stops the training flow: missing credentials, network errors, rate limits or a failed repair all fall back to the local dictionary.

A word is sent to a model only after the user chooses the online-generation action. The API key is read from the server environment and is never embedded in the page, saved in a snapshot or written to experiment records.

## Poking the API

```http
POST /api/v1/generate-item
Content-Type: application/json

{"word":"research"}
```

A successful response contains the learning `item` and its `provenance`. The older `GET /define?word=...` route is still available so the original interface keeps working.

Optional configuration:

```text
OPENAI_API_KEY=...       # keep this on the server
OPENAI_MODEL=...         # defaults to gpt-5.6-luna
IELTS_PORT=8765
```

The online path uses the Responses API with `reasoning.effort=none`, `store=false`, and strict `json_schema` output for the schema and guarded versions. See the [model documentation](https://developers.openai.com/api/docs/models/gpt-5.6-luna) and [Responses API reference](https://developers.openai.com/api/reference/resources/responses/methods/create).

## Reproducing the work

Run the checks:

```bash
python -m unittest discover -s tests -v
python -m py_compile run.py research/pipeline.py
node --check app.js
```

Run a deliberately small API smoke experiment (3 words × 3 versions × 1 repeat):

```bash
python -m research.run_experiment
```

The full planned call is opt-in: `python -m research.run_experiment --limit 100 --repeats 3`. Frozen JSONL results can be recalculated with `python -m research.evaluate PATH_TO_JSONL`.

## Boundaries I care about

- Personal answer history, timings, keys, caches and browser state do not belong in the public repository.
- The original 99-word source is used without personal response fields; the old CSV remains local and untracked.
- Browser speech varies by browser, and OCR may download model data only when the user invokes it.
- Passing an automated check is not the same as having a correct definition or a natural example. Human review is still necessary.
- This is currently a working product and an unfinished experiment—not evidence of learning improvement or an expert-reviewed system.

## Citation and licence

Citation metadata is in [`CITATION.cff`](CITATION.cff). Released under the [MIT License](LICENSE).
