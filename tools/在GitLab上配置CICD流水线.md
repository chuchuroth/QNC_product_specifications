在GitLab上配置CI/CD流水线的核心，是在项目仓库的根目录下创建一个名为 `.gitlab-ci.yml` 的文件。GitLab会检测到这个文件，并按照其中的指令由**Runner**来执行自动化任务。

下面是标准化的步骤，主要流程分为**配置Runner**和**编写流水线配置文件**两步。

### 1. 准备工作：确保有可用的Runner
**Runner** 就像是负责干活的“工人”，负责执行你定义的Job。如果你使用的是 **SaaS版本的 GitLab (如 gitlab.com)**，官方提供了免费的共享Runner，可以直接使用，无需自己搭建。

如果你是自托管的GitLab，或者需要特定的环境（例如一台内网服务器），则需要注册一个自己的Runner：
- **安装**：在你的服务器上安装 `gitlab-runner`。
- **注册**：在**项目** -> **设置** -> **CI/CD** -> **Runner** 页面找到项目的注册Token，然后运行 `sudo gitlab-runner register` 命令，根据提示填写GitLab地址、Token、执行器（如 `shell` 或 `docker`）等信息即可。

### 2. 核心步骤：编写 .gitlab-ci.yml 文件
在项目根目录创建 `.gitlab-ci.yml` 文件，这是控制整个流水线的核心配置文件。

#### 2.1 第一个基础示例
下面是一个最简单的配置，它定义了三个阶段（构建、测试、部署），每个阶段包含一个对应的任务：

```yaml
# 定义流水线的阶段顺序
stages:
  - build
  - test
  - deploy

# 构建任务
build-job:
  stage: build      # 属于 build 阶段
  script:
    - echo "正在编译代码..."
    - echo "编译完成"

# 测试任务
test-job:
  stage: test       # 属于 test 阶段
  script:
    - echo "开始运行单元测试..."
    - echo "测试通过"

# 部署任务
deploy-job:
  stage: deploy     # 属于 deploy 阶段
  script:
    - echo "正在部署到服务器..."
  environment: production   # 指定部署环境为生产环境
```

#### 2.2 进阶配置：完善你的真实场景
实际项目中，通常需要更精细的控制：

**① 控制触发规则**
默认每次推送都会触发。你可以用 `only` 或 `rules` 指定只有在特定分支或事件时才运行。
```yaml
# 只有在 main 分支的合并请求事件时才运行
job:
  script: echo "运行中"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_COMMIT_BRANCH_NAME == "main"'
```

**② 使用缓存加速**
对于 Node.js、Java 等项目，依赖下载很耗时，可以使用 `cache` 来复用文件。下面的配置会将 `node_modules` 文件夹缓存起来，加快 `npm install` 的速度：
```yaml
cache:
  paths:
    - node_modules/

job:
  script:
    - npm install
```

**③ 保存产物**
构建后生成的二进制文件或静态文件（如 `dist` 文件夹），可以用 `artifacts` 传递给后续的任务。
```yaml
build-job:
  stage: build
  script:
    - npm run build
  artifacts:          # 定义产物
    paths:
      - dist/         # 保存 dist 文件夹
    expire_in: 1 week # 产物保留一周
```

**④ 手动触发**
对于部署到生产环境等敏感操作，可以设置为手动触发。
```yaml
deploy-prod:
  stage: deploy
  script:
    - echo "部署到生产环境"
  when: manual   # 需要在GitLab界面上手动点击执行
```

### 3. 实操流程
1.  **创建文件**：在项目根目录新建 `.gitlab-ci.yml`，将上述配置写入。
2.  **提交代码**：将文件 `git add`、`git commit`、`git push` 到仓库。
3.  **查看结果**：提交后，进入GitLab项目页面，点击左侧菜单 **CI/CD** -> **流水线 (Pipelines)**，即可看到正在运行的流水线。点击状态图标可以查看详细的控制台日志。

### 4. 常用配置速查表
为了让你更快上手，这里整理了常用的配置关键字：

| 关键字 | 作用 | 使用场景举例 |
| :--- | :--- | :--- |
| `stages` | 定义任务执行顺序 | `- build` -> `- test` -> `- deploy` |
| `stage` | 将任务分配到指定阶段 | `stage: test` |
| `script` | **必须项**，要执行的Shell命令 | `- npm run test` |
| `only/rules` | 控制何时触发任务 | 只在 `main` 分支运行，或只在提MR时运行 |
| `cache` | 缓存依赖，加速构建 | 缓存 `node_modules` 或 `vendor` 文件夹 |
| `artifacts` | 在任务间传递文件，或下载 | 保存编译后的 `dist` 或 `jar` 包 |
| `when: manual` | 设为手动触发任务 | 部署到生产环境 |
| `environment` | 指定部署环境，便于追踪 | `environment: staging` |

### 5. 常见问题
-  **流水线卡住 (Pipeline stuck)**：这通常意味着没有可用的Runner。请检查 **设置** -> **CI/CD** -> **Runner** 页面，确认是否有活跃的Runner（带绿色圆圈）。
-  **敏感信息配置**：不要在脚本里硬编码密码或API密钥。请使用项目 **设置** -> **CI/CD** -> **变量 (Variables)** 来添加，然后在 `.gitlab-ci.yml` 中通过 `$VARIABLE_NAME` 调用。

这整个流程就是现代软件开发中**软件工程自动化**的典型实践，通过代码化配置（Pipeline as Code），大幅减少了手动操作带来的失误。
