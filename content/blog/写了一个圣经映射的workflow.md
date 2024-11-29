---
title: 写了一个圣经映射的workflow
date: 2024-11-29T05:23:01.444Z
---

代码是

import sys
import json
import pyperclip

# 加载圣经数据
with open("/Users/liyuge/【鲤鱼哥的文档】/Python Code/bible_output.json", "r", encoding="utf-8") as f:
    bible = json.load(f)

def parse_input(user_input):
    """
    解析用户输入，例如：约12:7 或 约12:7-8 或 约12:7-13:1
    支持 '：', ':' 作为分隔符
    """
    try:
        # 替换中文冒号和英文冒号
        user_input = user_input.replace('：', ':')
        
        if ":" not in user_input:
            raise ValueError("输入格式错误！请使用 '约12:7' 或 '约12:7-8' 或 '约12:7-13:1'")
        
        # 查找书卷名
        book = user_input[0]  # 获取书卷缩写 (如 "约")
        rest = user_input[1:]
        
        # 判断是否为多个章节
        if '-' in rest:
            chapter_verse_start, chapter_verse_end = rest.split('-')
            start_chapter, start_verse = chapter_verse_start.split(":")
            end_chapter, end_verse = chapter_verse_end.split(":")
            return book, int(start_chapter), int(start_verse), int(end_chapter), int(end_verse)
        else:
            chapter, verses = rest.split(":")
            start_verse, *end_verse = verses.split("-")
            end_verse = int(end_verse[0]) if end_verse else int(start_verse)
            return book, int(chapter), int(start_verse), end_verse

    except ValueError as e:
        print(f"解析错误：{e}")
        return None, None, None, None, None

def get_verses(book, start_chapter, start_verse, end_chapter, end_verse):
    results = []
    
    # 如果是同一章节的范围
    if start_chapter == end_chapter:
        for verse_num in range(start_verse, end_verse + 1):
            verse_text = bible.get(book, {}).get(str(start_chapter), {}).get(str(verse_num), None)
            if verse_text:
                results.append(f"【{book}{start_chapter}:{verse_num}】{verse_text}")
    else:
        # 如果跨多个章节
        for chapter in range(start_chapter, end_chapter + 1):
            start_v = start_verse if chapter == start_chapter else 1
            end_v = end_verse if chapter == end_chapter else 10000  # 假设每章最多有10000节
            for verse_num in range(start_v, end_v + 1):
                verse_text = bible.get(book, {}).get(str(chapter), {}).get(str(verse_num), None)
                if verse_text:
                    results.append(f"【{book}{chapter}:{verse_num}】{verse_text}")
    
    return "\n".join(results)

def main():
    # 获取用户输入
    user_input = sys.argv[1].strip() if len(sys.argv) > 1 else ""
    if not user_input:
        print("错误：未接收到输入！")
        return
    
    print(f"用户输入：{user_input}")  # 调试输出
    
    # 解析输入
    book, start_chapter, start_verse, end_chapter, end_verse = parse_input(user_input)
    if not book or not start_chapter or not start_verse or not end_chapter or not end_verse:
        print("输入格式错误！")
        return

    # 获取经文并复制到粘贴板
    result = get_verses(book, start_chapter, start_verse, end_chapter, end_verse)
    if result:
        pyperclip.copy(result)
        print("经文已复制到粘贴板：\n" + result)
    else:
        print("未找到匹配的经文。")

if __name__ == "__main__":
    main()


