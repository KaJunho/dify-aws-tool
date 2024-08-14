<p align="center">
    &nbsp<strong>简体中文</strong>&nbsp ｜ <a href="README.md"><strong>English</strong></a>&nbsp 
</p>
<br>

# Dify AWS Tool

## 简介
本仓库提供了一些示例代码,展示如何将 SageMaker Provider 和一些基于 AWS 服务的工具集成到 [Dify](https://github.com/langgenius/dify) 中。 

除了参考代码外,您还可以参考 Dify [官方指引](https://docs.dify.ai/guides/tools/quick-tool-integration) 获取更多信息。



## 前置条件

- Dify 环境

- AWS 账户和 AWS 使用经验

- 基本的 Linux 环境使用经验



## Assets 

***[注意]：欢迎大家贡献更多的workflow/sagemaker model/builtin tool, 可以fork本仓库提交merge request， 然后更新README.md， 自行在对应的表格新增一行***

#### 工作流 

| 名称                        | 描述                                        | Link                                                  | 依赖                            | 负责人                          |
| --------------------------- | ------------------------------------------- | ----------------------------------------------------- | ------------------------------- | ------------------------------- |
| Term_based_translate        | 集成了专词映射的翻译工作流                  | [DSL](./workflow/term_based_translation_workflow.yml) | Tool(Term_multilingual_mapping) | [ybalbert](ybalbert@amazon.com) |
| Code_translate              | 不同代码种类之间的翻译工作流                | [DSL](./workflow/claude3_code_translation.yml)        | Tool(LambdaYamlToJson)                                | [binc](binc@amazon.com)         |
| Basic_RAG_Sample            | 最基础的RAG工作流示例，包含自定义rerank节点 | [DSL](basic_rag_sample.yml)                           | Tool(Rerank)                    | [ybalbert](ybalbert@amazon.com) |
| Andrewyng/translation-agent | 复刻吴恩达的tranlsate agent                 | [DSL](andrew_translation_agent.yml)                   |                                 | [ybalbert](ybalbert@amazon.com) |

#### 内置工具

| 工具名称                  | 工具类型 | 描述                                                         | 部署文档                                                     | 负责人                          |
| ------------------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------- |
| Rerank                    | PAAS     | 文本相似性排序                                               | [Notebook](https://github.com/aws-samples/dify-aws-tool/blob/main/notebook/bge-reranker-v2-m3-deploy.ipynb) | [ybalbert](ybalbert@amazon.com) |
| TTS                       | PAAS     | 语音合成                                                     | [Code](https://github.com/aws-samples/dify-aws-tool/tree/main/notebook/cosyvoice) | [ybalbert](ybalbert@amazon.com) |
| Bedrock Guardrails        | SAAS     | 文本审核工具，通过 Amazon Bedrock Guardrail 上提供的独立评估API ApplyGuardrail 来实现。 |                                                              | [amyli](amyli@amazon.com)       |
| Term_multilingual_mapping | PAAS     | 切词/获取专词映射                                            | [Repo](https://github.com/ybalbert001/dynamodb-rag/tree/translate) | [ybalbert](ybalbert@amazon.com) |
| Image Translation Tool    | PAAS     | 翻译图片上的文字                                             | Comming                                                      | [tanqy](tangqy@amazon.com)      |
| LambdaYamlToJson          | PAAS      | 将 YAML 转为 JSON                                         | [README](./notebook/deploy_lambda_yaml_to_json.md)           | [binc](binc@amazon.com)      |

#### 模型提供商

| 模型名称         | 模型类型            | 部署文档                                                     | 负责人                          |
| ---------------- | ------------------- | ------------------------------------------------------------ | ------------------------------- |
| Bge-m3-rerank-v2 | SageMaker\Rerank    | [Notebook](https://github.com/aws-samples/dify-aws-tool/blob/main/notebook/bge-reranker-v2-m3-deploy.ipynb) | [ybalbert](ybalbert@amazon.com) |
| Bge-embedding-m3 | SageMaker\Embedding | [Notebook](https://github.com/aws-samples/dify-aws-tool/blob/main/notebook/bge-embedding-m3-deploy.ipynb) | [ybalbert](ybalbert@amazon.com) |
| CosyVoice        | SageMaker\TTS       | [Code](https://github.com/aws-samples/dify-aws-tool/tree/main/notebook/cosyvoice) | [ybalbert](ybalbert@amazon.com) |
| SenseVoice       | SageMaker\ASR       | [Notebook](https://github.com/aws-samples/dify-aws-tool/blob/main/notebook/funasr-deploy.ipynb) | [ybalbert](ybalbert@amazon.com) |



## 安装方法

***下面的脚本仅仅为了集成 SageMaker Model_provider 和 AWS Builtin Tools, 你可以从界面自行导入workflow。 SageMaker 模型供应商已经被集成到Dify v0.6.15***

1. 设置变量
   ```bash
   dify_path=/home/ec2-user/dify #Please set the correct dify install path
   tag=aws
   ```

2. 下载代码
   ```bash
   cd /home/ec2-user/
   git clone https://github.com/aws-samples/dify-aws-tool.git
   ```
   
3. 安装代码
   ```bash
   # 请注意，很多模型和工具已经被默认集成到Dify，无需额外安装
   mv ./dify-aws-tool/builtin_tools/aws ${dify_path}/api/core/tools/provider/builtin/
   mv ./dify-aws-tool/model_provider/sagemaker ${dify_path}/api/core/model_runtime/model_providers/
   ```
   
4. 构建新镜像

   ```
   cd ${dify_path}/api
   sudo docker build -t dify-api:${tag} .
   ```

5. 为api和worker服务指定新镜像

   ```diff
   # 修改docker/docker-compose.yaml, 请参考下面的diff
   diff --git a/docker/docker-compose.yaml b/docker/docker-compose.yaml
   index cffaa5a6a..38538e5ca 100644
   --- a/docker/docker-compose.yaml
   +++ b/docker/docker-compose.yaml
   @@ -177,7 +177,7 @@ x-shared-env: &shared-api-worker-env
    services:
      # API service
      api:
   -    image: langgenius/dify-api:0.6.14
   +    image: dify-api:aws
        restart: always
        environment:
          # Use the shared environment variables.
   @@ -197,7 +197,7 @@ services:
      # worker service
      # The Celery worker for processing the queue.
      worker:
   -    image: langgenius/dify-api:0.6.14
   +    image: dify-api:aws
        restart: always
        environment:
          # Use the shared environment variables.
   ```

5. 重启Dify
   ```bash
   cd ${dify_path}/docker/
   sudo docker-compose down
   sudo docker-compose up -d
   ```



## 如何部署SageMaker推理端点

如果您想将您的 Embedding/Rerank 模型添加到 Dify Sagemaker Model Provider,您应该首先在 Amazon SageMaker 中自行部署它们。详细参见[指引](./notebook/how_to_deploy.md).



## 目标受众
- Dify / AWS 用户
- 生成式 AI 开发者