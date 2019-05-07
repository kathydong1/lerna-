lerna
1 概要
lerna是GitHub上面开源的一款js代码库管理软件， 用来对一系列相互耦合比较大、又相互独立的js git库进行管理。解决各个库之间修改混乱、难以跟踪的问题。lerna可以优化这种情形下的工作流。
对于一些功能比较全的库，我们往往会把各个小功能拆分成独立的npm库，他们直接有比较强的依赖关系。比如：Babel、React等开源代码都是按照这种方式进行处理的。


2 代码库结构
lerna管理的库文件结构类似如下这样
my-lerna-repo/
  package.json
  packages/
    package-1/
      package.json
    package-2/
      package.json
      

3 lerna主要做了什么
通过lerna的命令lerna bootstrap 将会把代码库进行link。
通过lerna publish发布最新改动的库


@ 首先，初始化lerna库
cd lerna-repo
lerna init
@ 默认会创建lerna.json和packages目录, 如下
lerna-repo/
  packages/
  package.json
  lerna.json

4 lerna如何工作的  --------默认lerna有两种管理模式， 固定模式和独立模式
 @ 固定模式
固定模式，通过lerna.json的版本进行版本管理。当你执行lerna publish命令时， 如果距离上次发布只修改了一个模块，将会更新对应模块的版本到新的版本号，然后你可以只发布修改的库。
这种模式也是Babel使用的方式。如果你希望所有的版本一起变更， 可以更新minor版本号，这样会导致所有的模块都更新版本。
 @ 独立模式
独立模式，init的时候需要设置选项 --independent. 独立模式允许管理者对每个库单独改变版本号，每次发布的时候，你需要为每个改动的库指定版本号。这种情况下， lerna.json的版本号不会变化了， 默认为independent。

5 lerna.json解析
{
  "version": "1.1.3",
  "npmClient": "npm",
  "command": {
    "publish": {
      "ignoreChanges": [
        "ignored-file",
        "*.md"
      ]
    },
    "bootstrap": {
      "ignore": "component-*",
      "npmClientArgs": ["--no-package-lock"]      
    }
  },
  "packages": ["packages/*"]
}


version , 当前库的版本
npmClient , 允许指定命令使用的client， 默认是 npm， 可以设置成 yarn
command.publish.ignoreChanges ， 可以指定那些目录或者文件的变更不会被publish
command.bootstrap.ignore ， 指定不受 bootstrap 命令影响的包
command.bootstrap.npmClientArgs ， 指定默认传给 lerna bootstrap 命令的参数
command.bootstrap.scope ， 指定那些包会受 lerna bootstrap 命令影响
packages ， 指定包所在的目录

6 命令行
----lerna publish 发布新的库版本
@ lerna publish  // 发布最新commit的修改
@ lerna publish <commit-id> // 发布指定commit-id的代码
                参数 @canary 可以用来独立发布每个commit，不打tag
                        lerna publish --canary
                        # 1.0.0 => 1.0.1-alpha.0+${SHA} of packages changed since the previous commit
                        # a subsequent canary publish will yield 1.0.1-alpha.1+${SHA}, etc

                        lerna publish --canary --preid beta
                        # 1.0.0 => 1.0.1-beta.0+${SHA}

                        # The following are equivalent:
                        lerna publish --canary minor
                        lerna publish --canary preminor
                        # 1.0.0 => 1.1.0-alpha.0+${SHA}
                      @--npm-client 默认npm
                      @ --npm-tag    为发布的版本添加 dist-tag
                      @ --no-verify-access   不进行用户发布的权限校验
                      @ --registry <url>   指定registry
                      @ --yes      用于ci自动输入 yes
                      
------lerna version    这个命令 识别出修改的包 --> 创建新的版本号 --> 修改package.json --> 提交修改 打上版本的tag --> 推送到git上。
                      @ --allow-branch <glob>
                      设置git上的哪些分支允许执行 lerna version 命令， 也可以在lerna.json中设置
                      {
                        "command": {
                          "publish": {
                            "allowBranch": "master"
                          }
                        }
                      }

                      多个
                      {
                        "command": {
                          "publish": {
                            "allowBranch": ["master", "feature/*"]
                          }
                        }
                      }

                      @ --amend    把version的修改都合并到前一个commit， 而且不会推送哦。
                      @ --commit-hooks   执行对应的commit-hook， 默认true
                      @ --conventional-commits  使用了这个选项， lerna会收集日志， 自动生成 CHANGELOG
                      @ --changelog-preset 修改changelog生成插件， 默认是 angular
                      @ --force-publish 强制更改所有包的版本， 不管有没有修改
                      @ --ignore-changes 忽略检查某些文件的修改    
                            lerna version --ignore-changes '**/*.md' '**/__tests__/**'
                            也可以在lerna.json中配置
                            {
                              "ignoreChanges": [
                                "**/__fixtures__/**",
                                "**/__tests__/**",
                                "**/*.md"
                              ]
                            }

                      @ --git-remote <name>   把修改推送到其它的源， 而不是origin
                      @ --git-tag-version 添加git的tag， 默认true
                      @ --message <msg> 指定提交信息， 而不是自动生成的log
                          也可以在lerna.json配置
                          {
                            "command": {
                              "publish": {
                                "message": "chore(release): publish %s"
                              }
                            }
                          }
-------lerna bootstrap    bootstrap作了如下工作
                          为每个包安装依赖
                          链接相互依赖的库到具体的目录
                          执行 npm run prepublish
                          执行 npm run prepare
                          可以通过 -- 后添加选项, 设置npm cient的参数
                          lerna bootstrap -- --production --no-optional
                          也可以在lerna.json中配置
                          {
                            ...
                            "npmClient": "yarn",
                            "npmClientArgs": ["--production", "--no-optional"]
                          }

                          @--hoist 这个选项，会把共同依赖的库安装到根目录的node_modules下， 统一版本
                          @--ignore 忽略部分目录， 不进行安装依赖
                                    lerna bootstrap --ignore component-*
                                    也可以在lerna.json中配置
                                    {
                                      "version": "0.0.0",
                                      "command": {
                                        "bootstrap": {
                                          "ignore": "component-*"
                                        }
                                      }
                                    }  
                           @--ignore-scripts 不执行声明周期脚本命令， 比如 prepare
                           @--registry <url> 指定registry
                           @--npm-client 指定安装用的npm client
                                  lerna bootstrap --npm-client=yarn
                                  也可以在lerna.json中配置
                                  {
                                    ...
                                    "npmClient": "yarn"
                                  }

                           @--use-workspace   使用yarn workspace，
                           @--no-ci 默认调用 npm ci 替换 npm install , 使用选项修改设置
 
--------lerna list 列举当前lerna 库包含的包
                            @--json 显示信息为json格式
                            @--all 显示包含private的包
                            @--parseable 显示如下格式 <fullpath>:<name>:<version>[:flags..]
                            @--long显示更多的扩展信息
----------lerna changed 显示自上次relase tag以来有修改的包， 选项通 list

---------lerna diff 显示自上次relase tag以来有修改的包的差异， 执行 git diff

---------lerna exec 在每个包目录下执行任意命令
                            $ lerna exec -- <command> [..args] # runs the command in all packages
                            $ lerna exec -- rm -rf ./node_modules
                            $ lerna exec -- protractor conf.js
                            $ lerna exec -- npm view \$LERNA_PACKAGE_NAME
                            $ lerna exec -- node \$LERNA_ROOT_PATH/scripts/some-script.js
                            @ --concurrency 默认命令时并行执行的， 我们可以设置并发量为1     lerna exec --concurrency 1 -- ls -la
                            @ --scope   lerna exec --scope my-component -- ls -la
                            @ --stream 交叉并行输出结果  lerna exec --stream -- babel src -d lib
                            @ --parallel
                            @ --no-bail 默认lerna exec，如果有命令返回了非0， 则会停止执行， 通过设置这个参数 忽略返回非0， 继续执行其它命令
------------lerna run 执行每个包package.json中的脚本命令
                            $ lerna run <script> -- [..args] # runs npm run my-script in all packages that have it
                            $ lerna run test
                            $ lerna run build

                            # watch all packages and transpile on change, streaming prefixed output
                            $ lerna run --parallel watch


------------lerna init   创建一个新的lerna库或者是更新lerna版本，修改package.json中lerna版本，创建lerna.json
              --independent 独立模式
              --exeact

------------lerna add 添加一个包的版本为各个包的依赖
               lerna add <package>[@version] [--dev] [--exact]

-------------lerna clean 删除各个包下的node_modules

-------------lerna import 导入指定git仓库的包作为lerna管理的包
               lerna import <path-to-external-repository>
              @--flatten   如果有merge冲突， 用户可以使用这个选项所谓单独的commit
                  $ lerna import ~/Product --flatten
              @--dest    可以指定导入的目录(lerna.json中设定的目录)
                  $ lerna import ~/Product --dest=utilities
  
--------------lerna link 链接互相引用的库

--------------lerna create 新建包
              lerna create <name> [loc]

              Create a new lerna-managed package

              Positionals:
                name  The package name (including scope), which must be locally unique _and_
                      publicly available                                   [string] [required]
                loc   A custom package location, defaulting to the first configured package
                      location                                                        [string]

              Command Options:
                --access        When using a scope, set publishConfig.access value
                                           [choices: "public", "restricted"] [default: public]
                --bin           Package has an executable. Customize with --bin
                                <executableName>                             [default: <name>]
                --description   Package description                                   [string]
                --dependencies  A list of package dependencies                         [array]
                --es-module     Initialize a transpiled ES Module
                --homepage      The package homepage, defaulting to a subpath of the root
                                pkg.homepage                                          [string]
                --keywords      A list of package keywords                             [array]
                --license       The desired package license (SPDX identifier)   [default: ISC]
                --private       Make the new package private, never published
                --registry      Configure the package's publishConfig.registry        [string]
                --tag           Configure the package's publishConfig.tag             [string]
                --yes           Skip all prompts, accepting default values


              参考： https://github.com/lerna/lerna











lerna changed
显示自上次relase tag以来有修改的包， 选项通 list
lerna diff
显示自上次relase tag以来有修改的包的差异， 执行 git diff
lerna exec
在每个包目录下执行任意命令
使用示例
$ lerna exec -- <command> [..args] # runs the command in all packages
$ lerna exec -- rm -rf ./node_modules
$ lerna exec -- protractor conf.js
$ lerna exec -- npm view \$LERNA_PACKAGE_NAME
$ lerna exec -- node \$LERNA_ROOT_PATH/scripts/some-script.js

Options
--concurrency
默认命令时并行执行的， 我们可以设置并发量为1
lerna exec --concurrency 1 -- ls -la
--scope
lerna exec --scope my-component -- ls -la
--stream
交叉并行输出结果
lerna exec --stream -- babel src -d lib
--parallel
--no-bail
默认lerna exec，如果有命令返回了非0， 则会停止执行， 通过设置这个参数 忽略返回非0， 继续执行其它命令
lerna run
执行每个包package.json中的脚本命令
$ lerna run <script> -- [..args] # runs npm run my-script in all packages that have it
$ lerna run test
$ lerna run build

# watch all packages and transpile on change, streaming prefixed output
$ lerna run --parallel watch

