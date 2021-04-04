# learn-standard-version

之前做项目的时候都没在意过项目版本,最近自己写项目碰到版本的问题,目前 npm 版本管理工具用的比较多还是 standard-version , 这个仓库主要记录学习的笔记

## standard-version

standard-version 是一款遵循语义化版本（ semver）和 commit message 标准规范 的版本和 changlog 自动化工具。通常情况线下，我们会在 master 分支进行如下的版本发布操作：

1. git pull origin master
2. 根据 pacakage.json 中的 version 更新版本号，更新 changelog
3. git add -A, 然后 git commit
4. git tag 打版本操作
5. push 版本 tag 和 master 分支到仓库

其中2，3，4则是 standard-version 工具会自动完成的工作，配合本地的 shell 脚本，则可以自动完成一系列版本发布的工作了。

## 安装 & 使用

在这里我仍然推荐的全局安装：
```bash
$ npm install -g standard-version
# 或者
$ npm install --save-dev standard-version
# 执行：
# Help standard-version --help
$ standard-version

```

执行 standard-version 命令，我们会在控制台看到整个执行流程的 log 信息，在这里几个常用的参数需要注意下:

## --release-as, -r 指定版本号

默认情况下，工具会自动根据 主版本（major）,次版本（ minor） or 修订版（patch） 规则生成版本号，例如如果你package.json 中的version 为 1.0.0, 那么执行后版本号则是：1.0.1。自定义可以通过：

```

$ standard-version -r minor
# output 1.1.0
$ standard-version -r 2.0.0
# output 2.0.0
$ standard-version -r 2.0.0-test
# output 2.0.0-test

```

需要注意的是，这里的版本名称不是随便的字符，而是需要遵循语义化版本（ semver） 规范的

## --prerelease, -p 预发版本命名
用来生成预发版本, 如果当期的版本号是 2.0.0，例如

```bash
$ standard-version --prerelease alpha
# output 2.0.0-alpha.0

--tag-prefix, -t 版本 tag 前缀
用来给生成 tag 标签添加前缀，例如如果前版本号为 2.0.0，则：

$ standard-version --tag-prefix "stable-"
# output tag: stable-v2.0.0
```

以上这几个参数可能我们用的比较多，还有其他选项可以通过 standard-version --help查看。

集成 npm
最后记得把命令集成到 npm package.json的 scripts 中, 并配合 shell 脚本使用, 如下：

```json
"scripts": {
"release": "./scripts/release.sh",
"changelog": "conventional-changelog -p angular -i CHANGELOG.md -s -r 0 && git add CHANGELOG.md && npm run changeissueurl",
"changeissueurl": "replace 'https://github.com/myproject/issues/' 'https://redmine.example.com/' CHANGELOG.md"
}

```

配置好后使用 npm run 执行发布

```bash
$ npm run release
```

添加 release.sh 脚本：

```shell

#!/bin/bash

while [[ "$#" > 0 ]]; do case $1 in
  -r|--release) release="$2"; shift;;
  -b|--branch) branch="$2"; shift;;
  *) echo "Unknown parameter passed: $1"; exit 1;;
esac; shift; done

# Default as minor, the argument major, minor or patch: 
if [ -z "$release" ]; then
    release="minor";
fi

# Default release branch is master 
if [ -z "$branch" ] ; then
    branch="master"; 
fi;


echo "Branch is $branch"
echo "Release as $release"

# Tag prefix
prefix="prefix_v"

git pull origin $branch
echo "Current pull origin $branch."

# Generate version number and tag
standard-version -r $release --tag-prefix $prefix --infile CHANGELOG.md

git push --follow-tags origin $branch

echo "Release finished."

```

上面的脚本只是做了简单的分支 pull, 执行 standard-version 和最后的版本 push 工作，如果要做一些定制化的执行参数，则需要做定制修改了。

## Permission denied

添加 release 脚本的时候 可能会权限不够需要执行下面命令

```bash
$ sudo chmod -R 750 某一目录
```



## 参考

- [standard-version](https://github.com/conventional-changelog/standard-version)
- [git commit 、CHANGELOG 和版本发布的标准自动化
](https://zhuanlan.zhihu.com/p/51894196)
- [出现Permission denied的解决办法
](https://blog.csdn.net/qq_16525279/article/details/80245350)