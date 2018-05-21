# VIM操作速记


| 命令 | 快捷键 | 操作说明 |
| --- | --- | --- |
|  | Ctrl-w w | 光标在窗口之间切换 |
|  | Ctrl-w h, j, k, l | 光标在窗口之间切换 |
|  | Ctrl-w r | 切换当前窗口布局位置 |
|  | Ctrl-w p | 切换到前一个窗口 | 
| :help table |  | tab使用帮助 |
| :tabp | gt | 前一个tab页 |
| :tabn | gT | 后一个tab页 |
| :tabc |  | 关闭当前tab |
| :tabo |  | 关闭其他tab |
| :tabs |  | 查看所有打开的tab |
| :split | | 窗口横向拆分 |
| :vsplit | | 窗口纵向拆分 |
|  | Ctrl-w = | 让所有窗口均分 |
| :resize (-/+)n | Ctrl-w -/+ | 当前窗口减少/增加一（n）行 |
| :vertical resize (+/-)n | Ctrl-w >/< | 当前窗口增加、减少一（n）列 |



## nerdtree


| 命令 | 快捷键 | 操作说明 |
| --- | --- | --- |
| :NERDTreeToggle | Ctrl-n | 打开、关闭左侧导航栏 |
|  | t | 在新的Tab中打开文件 |
|   | i | 横向分屏打开文件 |
|  | s | 纵向分屏打开文件 |
|  | p | 跳到当前节点的父节点 |
|  | P | 跳到当前节点的根节点 |

文档：https://github.com/scrooloose/nerdtree/blob/master/doc/NERDTree.txt

## vim-go


| 命令 | 快捷键 | 操作说明 |
| --- | --- | --- |
| :GoDef | Ctrl-] | 转到定义 |
| :GoDefStack [number] |   | 跳转栈，不指定number，则展示栈列表 |
| :GoDefStackClear |   | 清空跳转栈 |
| :GoDefPop [count] | Ctrl-t | 定义跳回 |
| :GoDeps |  | 显示当前包的依赖 |
| :GoUpdateBinaries [binaries] |  | 下载并更新已经安装的go tools |

文档：https://github.com/fatih/vim-go/blob/master/doc/vim-go.txt

## majutsushi/tagbar


| 命令 | 快捷键 | 操作说明 |
| --- | --- | --- |
| :Tagbar | Ctrl-\ | 打开标签树 |

文档：https://github.com/majutsushi/tagbar


