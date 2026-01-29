---
date: 2025-10-09
tags:
  - rn
  - fronted-end
---

## 1. 创建react native项目

### 命令: 
```shell
yarn create expo-app 项目名 -t 模版
```
### 精简命令:
#### 把如下代码添加到.bash_profile文件,运行完source .bash_profile后,即可用rn 项目名 模版来创建react native项目
```Ïshell
rn() {
    local project_name="$1"
    local template="$2"
    
    # 常用模板列表
    local common_templates=("tabs" "blank" "bare-minimum" "navigation-template")
    
    if [ -z "$project_name" ] || [ -z "$template" ]; then
        echo "用法: rn <项目名> <模板名>"
        echo "常用模板: ${common_templates[*]}"
        return 1
    fi
    
    echo "🚀 创建 React Native 项目: $project_name"
    echo "📁 使用模板: $template"
    yarn create expo-app "$project_name" -t "$template"
}
```
### 模版参数描述: [详情](https://docs.expo.dev/more/create-expo/#--template)



