# Windows C盘深度清理工程日志：从爆红到满血复活

> **项目背景**：Windows 11 (200GB C盘)  
> **初始状态**：可用空间仅 17.8 GB (全线飘红)  
> **最终状态**：可用空间 62.6 GB (健康蓝区)  
> **核心目标**：安全释放空间，建立防复发机制。

---

## 清理前后对比

**🔴 清理前：C盘爆红，仅剩 17.8 GB 可用空间**

<img width="600" alt="清理前初始状态" src="https://github.com/user-attachments/assets/d066fe28-a41c-4198-a6a5-9e6f6b880fa5" />

*清理前的绝望时刻：C盘爆红，仅剩 17.8GB。*

**🟢 清理后：WizTree 扫描确认，C盘可用空间 62.6 GB**

<img width="600" alt="清理后最终状态" src="https://github.com/user-attachments/assets/0e35dc43-3ec9-428d-87ec-40e467dc6fdf" />

*清理后的胜利时刻：可用空间恢复到 62.6GB。*

---

## 1. 前言：为什么普通的“磁盘清理”没用？

半年前我曾尝试帮朋友清理C盘，仅靠 Windows 自带的“磁盘清理”和卸载软件，收效甚微。今天在排查自己的电脑时，终于找到了症结所在：**真正的空间杀手不在明面，而隐藏在软件缓存、系统隐藏文件（如休眠文件、虚拟内存）以及旧版本的残留中。** 本笔记记录了完整的排查逻辑与实战指令。

## 2. 基础环境排查

<img width="300" alt="磁盘清理截图" src="https://github.com/user-attachments/assets/aeaa69f1-178e-4d2d-82b4-8d68c29e0fdc" />

*   **操作**：`Win + R` 输入 `cleanmgr` -> 选择 C 盘 -> 点击“清理系统文件”（关键！）。
*   **总结**：此步骤仅清理出了 39.4 MB，属于“表面清理”（因为在这之前我按相同的方式操作了一次了）。

<img width="300" alt="系统存储全景图" src="https://github.com/user-attachments/assets/a53621fe-e891-4129-90cf-b6facd6f9e97" />

*   **操作**：设置 -> 系统 -> 存储。
*   **发现**：“安装的应用”占 95.6 GB，“系统和保留”占 61.6 GB。需要深入底层。

## 3. 底层深挖：手动排查、PowerShell 与核心文件处理

### 3.1 手动排查隐藏文件（揪出战地缓存）

<img width="500" alt="C盘开启隐藏文件前后对比" src="https://github.com/user-attachments/assets/f4245c5c-5fe5-4d2f-9389-58e01e917e96" />
<img width="500" alt="AppData目录下战地缓存截图" src="https://github.com/user-attachments/assets/ce60e0ab-ee80-46a1-899f-29f18818746c" />

*   **操作**：在C盘根目录（`C:\`）的资源管理器顶部“查看”菜单中，勾选“显示隐藏的项目”，对比开启前后多出来的隐藏文件和文件夹。
*   **深入排查**：随后进入 `C:\Users\覃朗\AppData\Local` 路径，继续查找大文件。

<img width="400" alt="战地缓存文件夹属性" src="https://github.com/user-attachments/assets/a642639a-afa7-4e41-b550-540e3484b4eb" />

*   **发现**：在该目录下找到了战地（Battlefield）残留的着色器缓存文件夹 `BattlefieldGameData.kin-live.Win32` (1.6 GB) 和 `BattlefieldGameData.kin-release.Win32` (3.2 GB)。
*   **处理**：由于游戏本体不在C盘，直接删除这两个文件夹，瞬间释放 **4.8 GB**。

<img width="200" alt="战地缓存最终属性" src="https://github.com/user-attachments/assets/d29f235f-d93a-48f1-8022-0c1804dbf1a5" />
<img width="200" alt="屏幕截图 2026-09-10 222253" src="https://github.com/user-attachments/assets/00aa38a6-244a-4b9f-a94b-b1bb0e66e871" />


### 3.2 PowerShell 深度扫描（抓出系统底层大件）

*   **操作**：`Win + X` -> 终端(管理员)，使用 `Get-ChildItem` 组合命令遍历文件夹真实大小。对于系统保护的隐藏文件夹，加上 `-Force` 参数进行穿透扫描。
*   **扫描结果**：
    *   通过单独执行命令测得：**`C:\Windows\Installer` 占 15.97 GB**（⚠️ **致命警告：这是系统安装缓存，用于软件卸载和修复，绝对不可手删！**）。
    *   普通扫描发现：`C:\Windows` 目录下，**`WinSxS` 占 13.23 GB**，**`System32` 占 9.18 GB**（系统命脉，不可动）。
    *   加上 `-Force` 参数后，穿透系统保护，扫描出隐藏文件夹：**`C:\Program Files\WindowsApps` 占 11.69 GB**（微软商店应用库）。
    *   根目录扫描发现：**`C:\$RECYCLE.BIN` (回收站) 占 5 GB**。

<img width="600" alt="Installer 扫描截图" src="https://github.com/user-attachments/assets/0adb654f-8bae-4fd5-9189-58c40b7f0545" />
<img width="600" alt="Windows 扫描截图" src="https://github.com/user-attachments/assets/f45cc9b1-edd6-45f7-9d7e-81833851ce92" />
<img width="600" alt="WindowsApps 扫描截图" src="https://github.com/user-attachments/assets/1a6ab390-6750-4bd3-a8e8-9e83f5b3eca5" />
<img width="600" alt="根目录全盘扫描" src="https://github.com/user-attachments/assets/2b65b69b-f453-4fe9-a71d-17ba29185bcd" />

*   **工程总结**：PowerShell 帮我们摸清了 C 盘的真实底牌。这些系统级隐藏文件夹虽然巨大，但绝对不能碰。真正能安全清理的，是那些软件缓存和游戏残留。

---

### 💡 工程备注：环境限制与 WizTree 的正确用法

**背景说明**：由于博主目前在俄罗斯，网络环境受限，无法直接下载 WizTree。因此，前两步（3.1 手动排查隐藏文件 和 3.2 PowerShell 深度扫描）是在无法使用可视化工具情况下的**备用排查方案**。如果你能顺利下载 WizTree，可以优先使用它，排查会非常直观。

#### 📥 WizTree 下载与使用指南（强烈推荐）

<img width="600" alt="WizTree 官网截图" src="https://github.com/user-attachments/assets/3e5b155f-4bd2-406b-a81a-f3200165d06e" />
<img width="600" alt="WizTree 解压截图" src="https://github.com/user-attachments/assets/46c49267-5cd0-4d44-8b47-7b9bd957110b" />

1. **下载**：前往 WizTree 官网 `diskanalyzer.com`，下载绿色便携版（Portable，是一个压缩包）。
2. **解压**：**千万不要解压到C盘**。在 D盘 新建一个文件夹（例如 `D:\WizTree`），将压缩包解压进去。
3. **管理员运行**：进入解压后的文件夹，找到 `WizTree64.exe`（64位系统），**右键 -> 以管理员身份运行**（这一步非常重要，否则无法扫描隐藏的系统文件）。
4. **扫描C盘**：在软件左上角选择 `C: [Windows]`，点击 `扫描 (Scan)` 按钮。只需 3 秒，界面就会生成一张巨大的**彩色方块图**，方块越大，文件越大。
5. **精准定位**：鼠标悬停在方块上即可看到具体文件路径。

<img width="600" alt="WizTree 彩色方块图" src="https://github.com/user-attachments/assets/31acdb5d-6651-4cce-9339-c3df09d8b359" />

*图：WizTree 的彩色方块图（当时找出 12.6GB 休眠文件，右下方，最大那块紫色）*

### 3.3 核心操作 A：关闭休眠文件（hiberfil.sys）

**🔴 清理前：休眠文件（hiberfil.sys）占据约 12.6 GB**

<img width="200" alt="休眠文件清理前截图" src="https://github.com/user-attachments/assets/9b802353-a404-4cc7-a8a1-f8c9b0b1dfcb" />

*   **操作**：管理员终端输入命令 `powercfg -h off`。
*   **原理**：休眠文件大小等于物理内存大小，关闭后瞬间释放十几GB空间（完全不影响正常关机和睡眠）。

> **⚠️ 操作前提**：此操作**仅适用于平时不使用“休眠”功能的用户**。
> 
> *   **“睡眠” vs “休眠”**：
>     *   **睡眠**：内存仍通电，数据不写入硬盘，唤醒极快，不受此操作影响。
>     *   **休眠**：将内存数据写入硬盘（即 `hiberfil.sys`），彻底断电后下次开机仍能恢复原状。
> *   如果你平时只用“关机”或“睡眠”，那么完全可以放心执行 `powercfg -h off` 删除休眠文件，能省下十几GB空间。
> *   如果你确实需要用“休眠”，请不要执行此命令。或者在需要时，用管理员终端输入 `powercfg -h on` 即可随时恢复该功能。

**🟢 清理后：再次运行 WizTree 扫描 C 盘，紫色休眠文件方块已消失**

<img width="600" alt="休眠文件清理后截图" src="https://github.com/user-attachments/assets/bdacef53-b5dd-4e66-a967-9a7f19220cde" />

### 3.4 核心操作 B：迁移虚拟内存（pagefile.sys）

**📖 为什么要迁移？**
`pagefile.sys`（虚拟内存文件）是系统在C盘根目录生成的一个隐藏大文件，通常默认等于物理内存的大小。因为它是隐藏且锁定的系统文件，直接删不掉，所以我们用“搬家”的方式，把它从C盘挪到D盘。

**🛠️ 操作步骤：**

1. **打开性能选项**：按 `Win + R`，输入 `sysdm.cpl` 并回车。在弹出的“系统属性”中，点击“高级”选项卡 -> 在“性能”区域点击“设置” -> 再点击“高级”选项卡。
<img width="200" alt="性能选项弹窗" src="https://github.com/user-attachments/assets/35623080-0993-4c44-881a-a53c4932879a" />

*图：在“性能选项”的高级选项卡中，点击虚拟内存区域的“更改(C)...”按钮。*

2. **迁移虚拟内存**：
   *   在“虚拟内存”窗口中，**取消勾选**最上方的“自动管理所有驱动器的分页文件大小”。
   *   选中 **C盘** -> 选择 **“无分页文件”** -> 点击 **“设置”**（弹警告点“是”）。
   *   选中 **D盘** -> 选择 **“系统管理的大小”** -> 点击 **“设置”**。
<img width="200" alt="虚拟内存设置界面" src="https://github.com/user-attachments/assets/b07ec210-da1d-41a9-824a-8e5ae9b3346b" />

*图：设置完成后的界面，C盘显示“无”，D盘显示“托管的系统”。*

3. **保存并重启**：一路点击“确定”关闭所有窗口，然后**重启电脑**。重启后，C盘那个隐形的 `pagefile.sys` 就会自动消失，空间才会真正释放出来。

> **⚠️ 避坑指南（踩坑经验）**：
> 如果你在点击“设置”按钮时感觉“没反应”，**那是正常的**。只要界面上 C 盘显示“无”，D 盘显示“托管的系统”，就说明已经配置成功了，不需要反复点击。直接点“确定”，重启电脑即可。

## 4. 核心重头戏：软件缓存无损迁移

### 4.1 微信（顺利迁移）
**操作**：微信设置 -> 账号与存储 -> 更改存储位置到 `D:\WeChat`（或你新建的文件夹）。
**结果**：成功搬家，C盘文档目录瞬间瘦身。
<img width="400" alt="微信账号与存储" src="https://github.com/user-attachments/assets/b63a2ea6-f455-4a1a-864d-b298ec1177c7" />

*图：微信存储设置，将保存路径从 C 盘更改至 D 盘。*

### 4.2 QQ（卸载重装与 OneDrive 陷阱）
**操作**：尝试在QQ的“存储管理”中修改路径，但遇到了权限问题。
**避坑**：QQ迁移时提示“云文件提供程序未运行（错误 0x8007016A）”，这是因为原路径在 OneDrive 中，文件变成了“云端占位符”，本地无法迁移。
**处理**：最终放弃迁移，**彻底卸载了 QQ**，并强行删除了 C 盘的 `Tencent Files` 残留文件，腾出了空间。（**注**：打算后续重新去官网下载 QQ 时，直接手动选择安装到 D 盘，重新登录后再把聊天记录路径设置在 D 盘，彻底避开 OneDrive 的坑）。

<img width="400" alt="QQ文件迁移报错" src="https://github.com/user-attachments/assets/006cfd04-b299-4e6e-8dd8-b96b6eaa72ec" />

*图：QQ文件迁移时发生错误。*

**最终解决方案（彻底根治）**：
即使重装QQ，依然反复提示“消息文件打开失败”。排查发现，QQ顽固地尝试把数据写入 `C:\Users\覃朗\OneDrive\文档\Tencent Files`，被 OneDrive 的按需下载机制死死卡住。于是采取了以下强制措施：

1. **彻底清场（清理所有旧残余）**：
   *   强制结束所有QQ/腾讯进程。
   *   删除 `AppData\Roaming\Tencent` 下的 `QQ`、`QQNT`、`TXSSO` 等残留。
   *   删除 `AppData\Local\Tencent` 下的缓存。
   *   强制删除 `Documents` 和 `OneDrive\文档` 下的 `Tencent Files` 文件夹。
   *   删除 `C:\Program Files\Tencent\QQ`（之前误安装在C盘的旧版本）。
   
<img width="200" alt="屏幕截图 2026-09-11 202039" src="https://github.com/user-attachments/assets/cb1d7f63-2ae4-49d5-840a-5fd2370f5241" />

*图：清理 Roaming 中的 QQ、QQNT、QQNTOpenSDK、QQTempSys、TXSSO文件夹。*

<img width="300" alt="屏幕截图 2026-09-11 202305" src="https://github.com/user-attachments/assets/09c9fb44-612d-4471-9c50-7c0cd9d46e2e" />

*图：强制删除系统 Documents 下的 Tencent Files。*

<img width="200" alt="屏幕截图 2026-09-11 203357" src="https://github.com/user-attachments/assets/2d8e0040-da4d-43e5-bbcc-c8834df55acc" />

*图：删除 OneDrive 文档下的 Tencent Files 残留。*

<img width="300" alt="屏幕截图 2026-09-11 203536" src="https://github.com/user-attachments/assets/76aadbaa-1abf-4d59-bfcf-75970ea29480" />

*图：删除之前误装在 C 盘 Program Files 下的旧 QQ 文件夹。*

2. **纯净安装**：去官网下载最新版，选择自定义安装到 **D盘全新路径**（如 `D:\TencentQQ`）。

3. **登录并第一时间改路径**：
   安装完成后扫码登录。成功进入QQ界面后，**第一时间**点开左下角三条杠 -> 设置 -> **存储管理**。
   *   此时“聊天消息默认保存到”还显示在 OneDrive，点击“更改存储路径”，把它改成纯本地的 **`D:\QQFiles`**（绝对避开 OneDrive）。
   
<img width="400" alt="屏幕截图 2026-09-11 204030" src="https://github.com/user-attachments/assets/a8d94465-0a7e-4b86-b7f5-e0f7a750f597" />

*图：进入QQ设置，将默认存储路径从 OneDrive 改为 D 盘纯本地路径。*

4. **执行数据迁移**：路径修改确认后，QQ 会自动将旧数据迁移到新路径下。
   
<img width="400" alt="屏幕截图 2026-09-11 204245" src="https://github.com/user-attachments/assets/dc85adeb-b675-493c-a192-10c1b8b35df1" />

*图：QQ正在将历史数据迁移至 D 盘新路径。*

5. **验证结果**：
   *   迁移完成后，重启QQ，确认不再报“消息文件打开失败”。
   *   去 D 盘看一眼 `D:\QQFiles\Tencent Files` 是否正常生成。
   
<img width="400" alt="屏幕截图 2026-09-11 204358" src="https://github.com/user-attachments/assets/549a3501-8df4-442a-8de2-131a1c4dd54d" />

*图：最终成功将聊天消息保存至 D 盘，彻底告别报错。*

### 4.3 WPS Office（清理了整整9个G！）
WPS 的清理过程稍微曲折，但也非常经典，一共分为5张关键截图：

1. **初始状态**：WPS 一度占用高达 **10.9 GB**。
<img width="400" alt="WPS清理前截图" src="https://github.com/user-attachments/assets/8d72cf32-4a1c-468e-9f54-7919712a9fec" />

*图：应用缓存 4.9 GB，云文档缓存 4.1 GB。*

2. **清理应用缓存**：点击“应用缓存”右侧的“立即清理”，此时释放了 4.9 GB。

3. **遭遇云缓存阻塞**：点击“云文档缓存”清理时，弹出提示“剩余 4.1 GB 暂无法清理”，因为里面包含“尚未上传成功的文件”。为了避免数据丢失，绝不能强行清理！
<img width="400" alt="WPS无法清理提示" src="https://github.com/user-attachments/assets/fc96d92a-93b6-42cf-b7b2-44147bed42e8" />

*图：WPS 提示有未上传的文件，无法直接清理。*

4. **迁移到D盘**：根据提示点击“前往迁移”，在弹出的文件选择框中，选中 D 盘新建的文件夹（如 `D:\WPS\WPS云文档缓存`）。
   
<img width="400" alt="WPS迁移路径选择" src="https://github.com/user-attachments/assets/14308691-2ab5-4a58-987d-4b5c1c1d60cc" />

*图：手动将云文档缓存迁移至 D 盘。*

<img width="400" alt="WPS正在迁移" src="https://github.com/user-attachments/assets/43e47291-82df-4792-a829-9834e8276df7" />

*图：正在迁移。*

5. **最终成果**：迁移完成并清理后，WPS 占用空间从 **10.9 GB 降至 1.9 GB**！
<img width="400" alt="WPS清理后截图" src="https://github.com/user-attachments/assets/8513a7a4-ae40-439c-82de-36c9d65364dc" />

*图：清理完成，占用降到 1.9 GB。*

### 4.4 补充：微软商店的陷阱与设置调整

**💡 重要结论：常用软件尽量不要从微软商店下载！**

在经历了 QQ 登录失败并重装的过程后，发现了一个大坑：**微软商店版的应用默认强制安装在 C 盘**（隐藏极深的 `C:\Program Files\WindowsApps` 文件夹），且经常受限于系统沙盒权限，导致数据迁移极其困难。加上部分国产软件（如 QQ）的商店版和官网版配置不互通，极易引发“消息文件打开失败”等诡异 Bug。

**✅ 正确的做法：**
去软件官网下载 `.exe` 安装包，安装时点击“自定义安装”，把路径改到 D 盘（如 `D:\Program Files\QQ`）。这样既独立干净，又能完美避开 C 盘爆满的风险。

**🛠️ 系统自带商店的“急救设置”：**
如果你确实需要用微软商店下载游戏（比如 Xbox Game Pass），一定要提前改好默认路径。

1. **打开设置**：在微软商店的“设置”中，找到“游戏安装选项”。
<img width="600" alt="屏幕截图 2026-09-11 232522" src="https://github.com/user-attachments/assets/dcbfdadf-2d7d-4771-9f73-50596c3911d3" />

*图：默认状态下，安装驱动器为 C 盘，安装文件夹为 `C:\XboxGames`。*

2. **更改驱动器**：点击“更改驱动器”下拉菜单，选择 **`D:`**。
<img width="400" alt="屏幕截图 2026-09-11 232755" src="https://github.com/user-attachments/assets/1acb3f95-1252-4bb3-bc2a-bb7804464d20" />

*图：在弹出的下拉菜单中，将驱动器从 C 切换到 D。*

3. **修改成功**：切换后系统会自动把路径变成 `D:\XboxGames`。确保“每次安装游戏询问我这些选项”的开关为 **开**。
<img width="400" alt="屏幕截图 2026-09-11 232812" src="https://github.com/user-attachments/assets/bf98281b-37a2-45b5-9578-510fd5ba6942" />

*图：修改成功，安装驱动器显示为 D 盘，文件夹变为 `D:\XboxGames`，一切准备就绪。*

> **💡 小提示**：截图下方还有“腾讯应用宝设置”。`腾讯应用宝` 和 `MobileAppEngine`（移动应用引擎）主要用于在电脑上运行安卓手机应用及手游。如果你平时不需要在电脑上运行手机 App，可以在“设置 -> 应用”里将它们卸载掉，这能额外腾出好几个 G 的空间！


## 5. 终极武器：WizTree 可视化分析

<img width="600" alt="WizTree全景截图" src="https://github.com/user-attachments/assets/232b75ee-5db4-465f-972a-16f9b07159bb" />

*   **工具**：WizTree（便携版，右键管理员运行）。
*   **作用**：3 秒扫描 C 盘，彩色方块图展示空间分布。
*   **警告**：`Windows`、`WinSxS` 等系统命脉，看着再大也绝对不许碰！

## 6. 防复发机制

| 策略 | 具体操作 | 目的 |
| :--- | :--- | :--- |
| **安装路径迁移** | 新装软件时，一律将 `C:\` 手动改为 `D:\`。 | 从源头截断 |
| **存储感知** | 设置 -> 系统 -> 存储 -> 开启“存储感知”，每周运行。 | 自动清理垃圾 |
| **文件重定向** | 桌面、下载、文档右键 -> 属性 -> 位置 -> 移动到 D 盘。 | 避免 C 盘被填满 |
| **缓存定期排雷** | 每季度检查一次微信、QQ、WPS、剪映等常用软件的存储设置。 | 防止缓存反弹 |

## 结语

其实，清理C盘并不难，难的是面对一团乱麻时找不到正确的方向。半年前帮朋友清理时，只是小打小闹清掉了一点皮毛；而这次，我花了整整一天时间，顺着蛛丝马迹，把 Windows 深藏的休眠文件、虚拟内存，以及微信、QQ、WPS 这些“缓存刺客”连根拔起。

这篇笔记里的每一步，我都替你踩过坑。QQ 诡异报错、WPS 云缓存阻塞这些让人抓狂的瞬间，我都一一截图记录了下来。如果你也是饱受“C盘红温”折磨的星人，请放心抄作业，路障我已经替你清除了。祝大家的电脑永不飘红，丝滑如初！

---

## 附录：PowerShell 命令行排查日志

由于 Windows 资源管理器无法准确计算隐藏文件夹的大小，我们在终端中使用 `Get-ChildItem` 组合命令进行了底层透视。以下是排查过程中用到的全部命令与结果节选。

<details>
<summary>点击展开：核心排查命令与执行结果（完整版）</summary>

### 1. 扫描用户数据目录（AppData）

**查询 Local 目录（存放软件缓存的主要位置）：**

```powershell
Get-ChildItem "C:\Users\覃朗（记得改成你的用户名）\AppData\Local" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 10
```

执行结果（节选）：

```text
Programs                    3.94
Microsoft                   3.28
Kingsoft                    2.94
JianyingPro                 2.04
KOOK                        1.69
```

**查询 Roaming 目录：**

```powershell
Get-ChildItem "C:\Users\覃朗（记得改成你的用户名）\AppData\Roaming" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 5
```

执行结果（节选）： `kingsoft 3.78, Tencent 3.59, WNS 2.05, .minecraft 1.79, baidu 1.41`

### 2. 扫描软件安装目录（Program Files）

**普通扫描（不显示隐藏文件夹）：**

```powershell
Get-ChildItem "C:\Program Files" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 8
```

执行结果（节选）： `MobileAppEngine 6.75, Huawei 4.86, Microsoft Office 4.3, Tencent 2.01`

```powershell
Get-ChildItem "C:\Program Files (x86)" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 8
```

执行结果（节选）： `Microsoft 5.65, Steam 1.24, LeiGod_Acc 0.88, Tencent 0.29`

**加上 -Force 参数穿透隐藏目录（抓出 WindowsApps）：**

```powershell
Get-ChildItem "C:\Program Files" -Directory -Force | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -Force -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 5
```

执行结果（节选）： `WindowsApps 11.69, MobileAppEngine 6.75, Huawei 4.86, Microsoft Office 4.3, Tencent 2.01`

### 3. 扫描系统级目录（ProgramData、Windows、Installer）

```powershell
Get-ChildItem "C:\ProgramData" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 8
```

执行结果（节选）： `Comms 3.39, Microsoft 1.54, Huawei 0.57`

```powershell
Get-ChildItem "C:\Windows" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 5
```

执行结果（节选）： `WinSxS 13.23, System32 9.18, SystemApps 1.32`
*(⚠️ 系统命脉，绝不能手删！)*

**单独测量 Windows\Installer 目录（普通扫描会遗漏）：**

```powershell
[math]::Round((Get-ChildItem "C:\Windows\Installer" -Recurse -File -Force -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)
```

执行结果： `15.97` GB
*(⚠️ 系统安装缓存，用于软件卸载和修复，绝对不可手删！)*

### 4. 全盘根目录扫描（穿透隐藏目录）

```powershell
Get-ChildItem "C:\" -Directory -Force | ForEach-Object { [PSCustomObject]@{Name=$_.FullName; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -Force -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 10
```

注：由于 C 盘根目录有系统权限保护的隐藏文件，该命令会报错“系统找不到指定的文件”，但仍能输出结果。
执行结果（节选）：

```text
C:\Windows                    50.64
C:\Program Files              32.21
C:\Program Files (x86)         9.03
C:\ProgramData                 6.11
C:\$RECYCLE.BIN                   5
C:\Recovery                    3.17
```

### 5. 执行清理操作

**关闭系统休眠功能，释放休眠文件 (hiberfil.sys) 空间：**

```powershell
powercfg -h off
# 瞬间释放约等于物理内存大小的空间（如 16GB 内存即释放 16GB），不影响正常关机和睡眠。
```

</details>
