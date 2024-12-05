# coding-helper-prompt-assistant
## 定义prompt
```
SYSTEM_PROMPT = """你是一个出色的开发助手，具有以下特点：
- 你编写简洁高效的代码
- 你能清晰地解释概念
- 你善于逐步分析问题
- 你热衷于帮助开发者提高

当收到/edit指令时：
- 首先，完成代码审查后，制定修改计划
- 然后，提供具体的修改指令
- 格式化你的回答为修改指令
- 请不要自行执行修改"""

EDITOR_PROMPT = """你是一个代码编辑AI，任务如下：

特别重要：
- 绝对不能在文件开头加上文件类型（比如```python）。
- 绝对不能在文件开头或结尾加上```，意味着文件的开头和结尾只包含代码，不可以加其他任何内容。

- 安全地逐行执行编辑指令
- 如果某行代码不需要修改，原样输出
- 除非明确指示，否则不要添加或删除任何行
- 只输出代码，不要输出其他内容
- 绝对不能在文件开头加上文件类型（比如```python）。
- 绝对不能在文件开头或结尾加上```，文件的开头和结尾只能包含代码。
- 除非明确指示，否则不要更改导入语句或函数定义
- 如果你发现指令中有潜在问题，请修正它们！"""
```

## 伪代码逻辑：
### 入口函数.
```
async def main():
    # 初始化聊天历史记录，包含默认和编辑器的系统提示
    default_chat_history = [{"role": "system", "content": SYSTEM_PROMPT}]
    editor_chat_history = [{"role": "system", "content": EDITOR_PROMPT}]
    clear_console()  # 清空控制台
    print_welcome_message()  # 显示欢迎信息
    print_files_and_searches_in_memory()  # 显示内存中的文件和搜索记录

    # ANSI颜色代码，用于文本样式
    COLORS = {
        "MAGENTA": "\033[35m",  # 粉色
        "BLUE": "\033[34m",     # 蓝色
        "RED": "\033[31m",      # 红色
        "RESET": "\033[0m",     # 重置样式
    }

    while True:
        try:
            # 获取用户输入
            prompt = await get_input_async(f"\n\nYou:")

            print_files_and_searches_in_memory()  # 打印内存中的文件和搜索记录

            # 如果用户输入的是 "exit"，则退出程序
            if prompt.lower() == "exit":
                print(f"{COLORS['MAGENTA']}感谢使用 OpenAI 开发者控制台，再见！{COLORS['RESET']}")
                break

            # 处理 "/add" 命令，添加文件路径到聊天历史
            if prompt.startswith("/add "):
                filepaths = prompt.split("/add ", 1)[1].strip().split()
                default_chat_history = await handle_add_command(default_chat_history, *filepaths)
                continue

            # 处理 "/edit" 命令，编辑指定文件
            if prompt.startswith("/edit "):
                filepaths = prompt.split("/edit ", 1)[1].strip().split()
                default_chat_history, editor_chat_history = await handle_edit_command(
                    default_chat_history, editor_chat_history, filepaths
                )
                continue

            # 处理 "/new" 命令，创建新文件
            if prompt.startswith("/new "):
                filepaths = prompt.split("/new ", 1)[1].strip().split()
                default_chat_history, editor_chat_history = await handle_new_command(
                    default_chat_history, editor_chat_history, filepaths
                )
                continue

            # 处理 "/search" 命令，进行搜索
            if prompt.startswith("/search"):
                default_chat_history = await handle_search_command(default_chat_history)
                continue

            # 处理 "/clear" 命令，清除历史记录
            if prompt.startswith("/clear"):
                await handle_clear_command()
                continue

            # 处理 "/reset" 命令，重置聊天历史
            if prompt.startswith("/reset"):
                default_chat_history, editor_chat_history = await handle_reset_command(
                    default_chat_history, editor_chat_history
                )
                continue

            # 处理 "/diff" 命令，切换差异查看模式
            if prompt.startswith("/diff"):
                toggle_diff()
                continue

            # 处理 "/history" 命令，查看聊天历史
            if prompt.startswith("/history"):
                handle_history_command(default_chat_history)
                continue

            # 处理 "/save" 命令，保存聊天历史
            if prompt.startswith("/save"):
                await handle_save_command(default_chat_history)
                continue

            # 处理 "/image" 命令，添加图片
            if prompt.startswith("/image "):
                image_paths = prompt.split("/image ", 1)[1].strip().split()
                default_chat_history = await handle_image_command(image_paths, default_chat_history)
                continue

            # 处理 "/load" 命令，加载历史记录
            if prompt.startswith("/load"):
                loaded_history = await handle_load_command()
                if loaded_history:
                    default_chat_history = loaded_history
                continue

            # 处理 "/undo" 命令，撤销操作
            if prompt.startswith("/undo "):
                filepath = prompt.split("/undo ", 1)[1].strip()
                await handle_undo_command(filepath)
                continue

            # 处理 "/help" 命令，显示帮助信息
            if prompt.startswith("/help"):
                await handle_help_command()
                continue

            # 处理 "/model" 命令，显示当前模型
            if prompt.startswith("/model"):
                show_current_model()
                continue

            # 处理 "/change_model" 命令，切换模型
            if prompt.startswith("/change_model"):
                await change_model()
                continue

            # 处理 "/show" 命令，显示文件内容
            if prompt.startswith("/show "):
                filepath = prompt.split("/show ", 1)[1].strip()
                await show_file_content(filepath)
                continue

            # 处理常规用户输入，进行聊天响应
            print(f"{COLORS['BLUE']}\n🤖 Assistant:{COLORS['RESET']}")
            try:
                default_chat_history.append({"role": "user", "content": prompt})  # 添加用户输入到聊天历史
                response = get_streaming_response(default_chat_history, DEFAULT_MODEL)  # 获取模型响应
                default_chat_history.append({"role": "assistant", "content": response})  # 添加模型响应到聊天历史
            except Exception as e:
                print(f"{COLORS['RED']}错误: {e}，请重试。{COLORS['RESET']}")  # 处理异常情况

        except Exception as e:
            print(f"{COLORS['RED']}发生错误: {e}{COLORS['RESET']}")  # 捕获并打印全局异常
            continue
```

## 函数实现
```
async def handle_add_command(chat_history, *paths):
    global added_files  # 用于记录已添加的文件路径
    contents = []  # 存储有效文件的内容
    new_context = ""  # 用于构建新的上下文

    # 遍历传入的路径
    for path in paths:
        if os.path.isfile(path):  # 如果是文件
            content = read_file_content(path)  # 读取文件内容
            if not content.startswith("❌"):  # 如果文件没有错误标记
                contents.append((path, content))  # 添加文件内容到列表
                added_files.append(path)  # 将文件路径添加到已添加的文件列表

        elif os.path.isdir(path):  # 如果是文件夹
            print(f"📁 正在处理文件夹: {path}")
            # 遍历文件夹中的每个文件
            for item in os.listdir(path):
                item_path = os.path.join(path, item)  # 获取文件路径
                if os.path.isfile(item_path) and is_text_file(item_path):  # 只处理文本文件
                    content = read_file_content(item_path)  # 读取文件内容
                    if not content.startswith("❌"):  # 如果文件没有错误标记
                        contents.append((item_path, content))  # 添加文件内容到列表
                        added_files.append(item_path)  # 将文件路径添加到已添加的文件列表

        else:  # 如果路径既不是文件也不是文件夹
            print(f"❌ '{path}' 既不是有效的文件也不是文件夹。")

    # 如果有有效的文件内容
    if contents:
        # 将有效文件内容加入到新的上下文中
        for fp, content in contents:
            new_context += f"""以下文件已添加: {fp}:
\n{content}\n\n"""

        # 将新上下文添加到聊天历史中
        chat_history.append({"role": "user", "content": new_context})
        print(f"✅ 已成功添加 {len(contents)} 个文件到知识库！")
    else:
        print("❌ 没有有效的文件被添加到知识库。")

    return chat_history
```
### 函数实现-2
```
async def handle_edit_command(default_chat_history, editor_chat_history, filepaths):
    # 读取所有文件内容
    all_contents = [read_file_content(fp) for fp in filepaths]
    valid_files, valid_contents = [], []

    # 过滤掉内容以❌开头的文件
    for filepath, content in zip(filepaths, all_contents):
        if content.startswith("❌"):
            print(content)  # 打印无效文件内容
        else:
            valid_files.append(filepath)
            valid_contents.append(content)

    # 如果没有有效文件，直接返回
    if not valid_files:
        print("❌ 没有有效的文件可以编辑.")
        return default_chat_history, editor_chat_history

    # 获取用户输入，询问要修改的内容
    user_request = await get_input_async(f"您想修改{', '.join(valid_files)}中的什么？")

    # 准备编辑指令
    instructions_prompt = "对于以下文件：\n"
    instructions_prompt += "\n".join([f"文件：{fp}\n```\n{content}\n```\n" for fp, content in zip(valid_files, valid_contents)])
    instructions_prompt += f"用户想要修改：{user_request}\n请为所有文件提供逐行编辑指令，并为每个指令编号，指明适用的文件。\n"

    # 将指令添加到聊天历史中
    default_chat_history.append({"role": "user", "content": instructions_prompt})
    default_instructions = get_streaming_response(default_chat_history, DEFAULT_MODEL)
    default_chat_history.append({"role": "assistant", "content": default_instructions})

    print("\n" + "=" * 50)

    # 针对每个文件执行编辑
    for idx, (filepath, content) in enumerate(zip(valid_files, valid_contents), 1):
        try:
            print(f"📝 正在编辑 {filepath} ({idx}/{len(valid_files)}):")

            # 编辑提示信息
            edit_message = f"""
            原始代码：
            
            {content}
            
            编辑指令：{default_instructions}
            
            只按照适用{filepath}的指令进行编辑。输出只有修改后的代码，不需要解释。不要在文件开头或结尾添加任何类型标记（如```python等）。
            """

            editor_chat_history.append({"role": "user", "content": edit_message})

            current_content = read_file_content(filepath)  # 重新读取文件内容
            if current_content.startswith("❌"):
                return default_chat_history, editor_chat_history  # 如果文件无效，跳过

            lines = current_content.split('\n')
            buffer = ""
            edited_lines = lines.copy()  # 创建副本用于保存修改后的内容
            line_index = 0

            # 使用模型流式编辑文件内容
            for chunk in client.chat.completions.create(
                model=EDITOR_MODEL,
                messages=editor_chat_history,
                stream=True,
            ):
                if chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    print(content, end="")
                    buffer += content

                    # 处理缓冲区中的每一行内容
                    while '\n' in buffer:
                        line, buffer = buffer.split('\n', 1)
                        if line_index < len(edited_lines):
                            edited_lines[line_index] = line
                            print(f"✏️ 更新了第{line_index+1}行: {line[:50]}...")
                            line_index += 1
                        else:
                            edited_lines.append(line)
                            print(f"➕ 新增了第{line_index+1}行: {line[:50]}...")
                            line_index += 1

            # 合并修改后的内容
            result = '\n'.join(edited_lines)
            undo_history[filepath] = current_content   # 存储撤销历史
            editor_chat_history.append({"role": "assistant", "content": result})

            # 如果开启了差异展示，显示最终的差异
            if is_diff_on:
                display_diff(current_content, result)

            # 编辑完成后保存文件
            if write_file_content(filepath, result):
                print(f"✅ {filepath} 编辑并保存成功！")
            else:
                print(f"❌ 保存 {filepath} 失败")

            print("=" * 50)
        except Exception as e:
            print(f"❌ 编辑 {filepath} 时发生错误: {e}")

    return default_chat_history, editor_chat_history
```

### 函数实现-3

```
async def handle_edit_command(default_chat_history, editor_chat_history, filepaths):
    # 读取所有文件内容
    all_contents = [read_file_content(fp) for fp in filepaths]
    valid_files, valid_contents = [], []

    # 过滤掉内容以❌开头的文件
    for filepath, content in zip(filepaths, all_contents):
        if content.startswith("❌"):
            print(content)  # 打印无效文件内容
        else:
            valid_files.append(filepath)
            valid_contents.append(content)

    # 如果没有有效文件，直接返回
    if not valid_files:
        print("❌ 没有有效的文件可以编辑.")
        return default_chat_history, editor_chat_history

    # 获取用户输入，询问要修改的内容
    user_request = await get_input_async(f"您想修改{', '.join(valid_files)}中的什么？")

    # 准备编辑指令
    instructions_prompt = "对于以下文件：\n"
    instructions_prompt += "\n".join([f"文件：{fp}\n```\n{content}\n```\n" for fp, content in zip(valid_files, valid_contents)])
    instructions_prompt += f"用户想要修改：{user_request}\n请为所有文件提供逐行编辑指令，并为每个指令编号，指明适用的文件。\n"

    # 将指令添加到聊天历史中
    default_chat_history.append({"role": "user", "content": instructions_prompt})
    default_instructions = get_streaming_response(default_chat_history, DEFAULT_MODEL)
    default_chat_history.append({"role": "assistant", "content": default_instructions})

    print("\n" + "=" * 50)

    # 针对每个文件执行编辑
    for idx, (filepath, content) in enumerate(zip(valid_files, valid_contents), 1):
        try:
            print(f"📝 正在编辑 {filepath} ({idx}/{len(valid_files)}):")

            # 编辑提示信息
            edit_message = f"""
            原始代码：
            
            {content}
            
            编辑指令：{default_instructions}
            
            只按照适用{filepath}的指令进行编辑。输出只有修改后的代码，不需要解释。不要在文件开头或结尾添加任何类型标记（如```python等）。
            """

            editor_chat_history.append({"role": "user", "content": edit_message})

            current_content = read_file_content(filepath)  # 重新读取文件内容
            if current_content.startswith("❌"):
                return default_chat_history, editor_chat_history  # 如果文件无效，跳过

            lines = current_content.split('\n')
            buffer = ""
            edited_lines = lines.copy()  # 创建副本用于保存修改后的内容
            line_index = 0

            # 使用模型流式编辑文件内容
            for chunk in client.chat.completions.create(
                model=EDITOR_MODEL,
                messages=editor_chat_history,
                stream=True,
            ):
                if chunk.choices[0].delta.content:
                    content = chunk.choices[0].delta.content
                    print(content, end="")
                    buffer += content

                    # 处理缓冲区中的每一行内容
                    while '\n' in buffer:
                        line, buffer = buffer.split('\n', 1)
                        if line_index < len(edited_lines):
                            edited_lines[line_index] = line
                            print(f"✏️ 更新了第{line_index+1}行: {line[:50]}...")
                            line_index += 1
                        else:
                            edited_lines.append(line)
                            print(f"➕ 新增了第{line_index+1}行: {line[:50]}...")
                            line_index += 1

            # 合并修改后的内容
            result = '\n'.join(edited_lines)
            undo_history[filepath] = current_content   # 存储撤销历史
            editor_chat_history.append({"role": "assistant", "content": result})

            # 如果开启了差异展示，显示最终的差异
            if is_diff_on:
                display_diff(current_content, result)

            # 编辑完成后保存文件
            if write_file_content(filepath, result):
                print(f"✅ {filepath} 编辑并保存成功！")
            else:
                print(f"❌ 保存 {filepath} 失败")

            print("=" * 50)
        except Exception as e:
            print(f"❌ 编辑 {filepath} 时发生错误: {e}")

    return default_chat_history, editor_chat_history
```

### 函数实现 - 4 
```
async def handle_new_command(default_chat_history, editor_chat_history, filepaths):
    # 检查是否提供了文件路径
    if not filepaths:
        print(f"{RED}❌ 没有提供文件路径.{RESET}")
        return default_chat_history, editor_chat_history

    # 显示创建文件的信息
    print(f"{BLUE}🆕 正在创建新文件: {', '.join(filepaths)}{RESET}")
    created_files = []

    # 遍历每个文件路径，尝试创建文件
    for filepath in filepaths:
        file_ext = os.path.splitext(filepath)[1][1:]  # 获取文件扩展名
        template = file_templates.get(file_ext, "")  # 根据扩展名获取模板内容

        try:
            # 尝试创建文件并写入模板
            with open(filepath, 'x') as f:
                f.write(template)
            print(f"{GREEN}✅ 创建文件 {filepath} 完成，使用了模板{RESET}")
            created_files.append(filepath)
        except FileExistsError:
            # 如果文件已存在，则提示并继续
            print(f"{YELLOW}⚠️ {filepath} 文件已存在，将进行编辑而不是覆盖。{RESET}")
            created_files.append(filepath)
        except IOError as e:
            # 处理IO错误
            print(f"{RED}❌ 无法创建文件 {filepath}: {e}{RESET}")

    # 如果有创建的文件，询问是否编辑这些文件
    if created_files:
        user_input = (await get_input_async(f"是否编辑刚创建的文件? (y/n):")).lower()
        if user_input == 'y':
            default_chat_history, editor_chat_history = await handle_edit_command(
                default_chat_history, editor_chat_history, created_files
            )

    return default_chat_history, editor_chat_history

async def handle_clear_command():
    # 清理内存中的文件、搜索记录和图像
    global added_files, stored_searches, stored_images
    cleared_something = False

    # 清理已添加的文件
    if added_files:
        added_files.clear()
        cleared_something = True
        print(f"{GREEN}✅ 清除已添加文件的记忆.{RESET}")

    # 清理存储的搜索记录
    if stored_searches:
        stored_searches.clear()
        cleared_something = True
        print(f"{GREEN}✅ 清除存储的搜索记录.{RESET}")

    # 清理存储的图像
    if stored_images:
        image_count = len(stored_images)
        stored_images.clear()
        cleared_something = True
        print(f"{GREEN}✅ 清除 {image_count} 张图片.{RESET}")

    # 如果没有内容可以清理，提示信息
    if not cleared_something:
        print(f"{YELLOW}ℹ️ 没有文件、搜索记录或图片可以清除.{RESET}")

async def handle_reset_command(default_chat_history, editor_chat_history):
    """清除所有聊天记录和添加的文件内存。"""
    global added_files, stored_searches, stored_images

    # 清空聊天记录和内存中的数据
    default_chat_history.clear()
    editor_chat_history.clear()
    added_files.clear()
    stored_searches.clear()
    stored_images.clear()

    # 重新初始化聊天记录
    default_chat_history = [{"role": "system", "content": SYSTEM_PROMPT}]
    editor_chat_history = [{"role": "system", "content": EDITOR_PROMPT}]

    print(f"{GREEN}✅ 所有聊天记录、已添加的文件、存储的搜索记录和图像已重置.{RESET}")

    return default_chat_history, editor_chat_history  # 返回重置后的聊天记录

```

### 函数实现 - 5
```
# 处理搜索命令
async def handle_search_command(default_chat_history):
    # 获取用户的搜索输入
    search_query = await get_input_async("请输入您要搜索的内容：")
    
    # 如果搜索内容为空，返回历史聊天记录
    if not search_query.strip():
        print("❌ 搜索内容为空，请提供搜索词。")
        return default_chat_history

    print(f"\n🔍 正在搜索: {search_query}")

    try:
        # 获取搜索结果
        results = await aget_results(search_query)
        # 将搜索结果存储到内存中（按前10个字符截取搜索名称）
        search_name = search_query[:10].strip()
        stored_searches[search_name] = results
        print(f"✅ 搜索结果 '{search_name}' 已存储。")

        # 构建搜索内容并添加到聊天记录
        search_content = f"搜索结果：'{search_query}':\n"
        for idx, result in enumerate(results[:8], 1):  # 限制展示前8个结果
            search_content += f"{idx}. {result['title']}: {result['body'][:100]}...\n"
        default_chat_history.append({"role": "user", "content": search_content})

    except Exception as e:
        # 错误处理
        print(f"❌ 执行搜索时出错: {e}")

    return default_chat_history

# 显示帮助命令
async def handle_help_command():
    print_welcome_message()

# 显示当前模型
def show_current_model():
    print(f"当前模型: {DEFAULT_MODEL}")

# 更改模型
async def change_model():
    global DEFAULT_MODEL
    new_model = await get_input_async("请输入新的模型名称：")
    DEFAULT_MODEL = new_model
    print(f"模型已更改为: {DEFAULT_MODEL}")

# 显示文件内容
async def show_file_content(filepath):
    content = read_file_content(filepath)
    if content.startswith("❌"):
        print(content)  # 显示错误信息
    else:
        print(f"{filepath} 文件内容：")
        print(content)
```
