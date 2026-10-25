---
title: 批量处理漫画jpg/png/gif为cbz格式
id: 01a0da29-f560-72eb-be54-d232e445729d
date: 2026-09-26 04:03:09
auther: squaly
featured_image: Pasted%20image%2020261024070944.png
featured_image_preview: Pasted%20image%2020261024070944.png
summary: 1. 创建通用脚本 例如放在： nano ~/manga_to_cbz.sh 完整复制下面内容：
permalink: /archives/toQwuF48
categories:
  - tools
tags:
  - manga
---

# 1. 创建通用脚本

例如放在：

```
nano ~/manga_to_cbz.sh
```

完整复制下面内容：

```
#!/usr/bin/env bash

# ============================================================
# Manga → CBZ Converter v5.0
#
# 支持：
#
# 方式 1：PDF 每卷一个文件
#   Manga Vol.01.pdf
#   Manga Vol.02.pdf
#   Manga Vol.03.pdf
#
# 方式 2：每卷一个目录
#   01/
#   02/
#   03/
#
# 方式 3：当前目录直接放图片
#   001.jpg
#   002.jpg
#   003.jpg
#
# 功能：
#   PDF → JPEG
#   JPG / JPEG / PNG / GIF / WEBP → JPEG
#   自动识别双页
#   双页阅读顺序：右 → 左
#   单页保持原页
#   自动生成 CBZ
#   已完成卷自动跳过
#   Ctrl+C 后可继续
#   失败时保留临时文件
#
# 依赖：
#   pdfinfo
#   pdftoppm
#   identify
#   convert
#   zip
#   unzip
# ============================================================

set -u
set -o pipefail

VERSION="5.0"

# ============================================================
# 配置
# ============================================================

BASE_DIR="$(pwd)"

OUTPUT_DIR="$BASE_DIR/cbz"
TMP_DIR="$BASE_DIR/.cbz_tmp"

# PDF DPI
PDF_DPI=300

# PDF → JPEG 质量
PDF_JPEG_QUALITY=95

# 图片 → JPEG 质量
JPEG_QUALITY=95

# 宽高比超过这个值才认为是双页
DOUBLE_PAGE_RATIO=1.80

# ============================================================
# 颜色
# ============================================================

if [ -t 1 ]; then
    RED='\033[0;31m'
    GREEN='\033[0;32m'
    YELLOW='\033[1;33m'
    BLUE='\033[0;34m'
    CYAN='\033[0;36m'
    RESET='\033[0m'
else
    RED=''
    GREEN=''
    YELLOW=''
    BLUE=''
    CYAN=''
    RESET=''
fi

# ============================================================
# 日志
# ============================================================

info() {
    echo -e "${BLUE}[INFO]${RESET} $*"
}

ok() {
    echo -e "${GREEN}[ OK ]${RESET} $*"
}

warn() {
    echo -e "${YELLOW}[WARN]${RESET} $*"
}

err() {
    echo -e "${RED}[ERR ]${RESET} $*" >&2
}

# ============================================================
# Ctrl+C
# ============================================================

interrupted() {

    echo
    echo
    warn "任务被中断。"
    warn "临时文件保留在："
    echo "  $TMP_DIR"
    echo
    warn "重新运行脚本即可继续。"

    exit 130
}

trap interrupted INT TERM

# ============================================================
# 检查依赖
# ============================================================

check_dependencies() {

    local missing=0

    echo
    info "检查依赖..."

    local cmd

    for cmd in \
        pdfinfo \
        pdftoppm \
        identify \
        convert \
        zip \
        unzip
    do

        if command -v "$cmd" >/dev/null 2>&1; then
            echo "    ✓ $cmd"
        else
            echo "    ✗ $cmd"
            missing=1
        fi

    done

    if [ "$missing" -ne 0 ]; then

        echo
        err "缺少必要程序。"
        echo
        echo "安装："
        echo
        echo "sudo apt install poppler-utils imagemagick zip unzip"
        echo

        exit 1
    fi

    echo
}

# ============================================================
# 判断是不是图片
# ============================================================

is_image() {

    local file="$1"

    case "${file,,}" in
        *.jpg)
            return 0
            ;;
        *.jpeg)
            return 0
            ;;
        *.png)
            return 0
            ;;
        *.gif)
            return 0
            ;;
        *.webp)
            return 0
            ;;
        *)
            return 1
            ;;
    esac
}

# ============================================================
# 获取图片尺寸
# ============================================================

get_size() {

    identify -format '%w %h' "$1" 2>/dev/null
}

# ============================================================
# 图片序号
# ============================================================

IMAGE_INDEX=0

# ============================================================
# 处理一张图片
#
# 如果是双页：
#
#   ┌───────────┬───────────┐
#   │    左     │    右     │
#   │   page2   │   page1   │
#   └───────────┴───────────┘
#
# 输出：
#
#   00001.jpg = 右
#   00002.jpg = 左
# ============================================================

process_image() {

    local source="$1"

    local size
    local width
    local height
    local ratio

    size="$(get_size "$source")"

    if [ -z "$size" ]; then
        err "无法读取图片：$(basename "$source")"
        return 1
    fi

    read -r width height <<< "$size"

    if [ -z "$width" ] || [ -z "$height" ]; then
        err "无法读取图片尺寸：$(basename "$source")"
        return 1
    fi

    ratio=$(awk \
        -v w="$width" \
        -v h="$height" \
        'BEGIN {printf "%.3f", w/h}')

    # ========================================================
    # 双页
    # ========================================================

    if awk \
        -v r="$ratio" \
        -v t="$DOUBLE_PAGE_RATIO" \
        'BEGIN {exit !(r >= t)}'
    then

        local half_width=$((width / 2))

        # -----------------------------
        # 右半页
        # -----------------------------

        IMAGE_INDEX=$((IMAGE_INDEX + 1))

        local right="$CURRENT_WORK/$(printf '%05d.jpg' "$IMAGE_INDEX")"

        if ! convert "$source" \
            -crop "${half_width}x${height}+${half_width}+0" \
            +repage \
            -quality "$JPEG_QUALITY" \
            "$right" \
            >/dev/null 2>&1
        then

            err "双页右半处理失败：$(basename "$source")"
            return 1
        fi

        # -----------------------------
        # 左半页
        # -----------------------------

        IMAGE_INDEX=$((IMAGE_INDEX + 1))

        local left="$CURRENT_WORK/$(printf '%05d.jpg' "$IMAGE_INDEX")"

        if ! convert "$source" \
            -crop "${half_width}x${height}+0+0" \
            +repage \
            -quality "$JPEG_QUALITY" \
            "$left" \
            >/dev/null 2>&1
        then

            err "双页左半处理失败：$(basename "$source")"
            return 1
        fi

    else

        # ====================================================
        # 普通单页
        # ====================================================

        IMAGE_INDEX=$((IMAGE_INDEX + 1))

        local output="$CURRENT_WORK/$(printf '%05d.jpg' "$IMAGE_INDEX")"

        if ! convert "$source" \
            -quality "$JPEG_QUALITY" \
            "$output" \
            >/dev/null 2>&1
        then

            err "图片转换失败：$(basename "$source")"
            return 1
        fi

    fi

    return 0
}

# ============================================================
# PDF 页数
# ============================================================

get_pdf_pages() {

    pdfinfo "$1" 2>/dev/null |
        awk '/^Pages:/ {print $2}'
}

# ============================================================
# 处理 PDF
# ============================================================

process_pdf() {

    local pdf="$1"

    local pages

    pages="$(get_pdf_pages "$pdf")"

    if [ -z "$pages" ]; then
        err "无法读取 PDF 页数：$(basename "$pdf")"
        return 1
    fi

    info "PDF：$(basename "$pdf")"
    info "PDF页数：$pages"

    local page
    local rendered

    for ((page=1; page<=pages; page++))
    do

        printf "\r    PDF %d/%d" "$page" "$pages"

        rendered="$CURRENT_WORK/__page.jpg"

        rm -f "$rendered"

        # ====================================================
        # PDF 单页渲染
        #
        # 明确使用：
        #
        # __page.jpg
        #
        # 不再依赖 pdftoppm 自动编号。
        # ====================================================

        if ! pdftoppm \
            -f "$page" \
            -l "$page" \
            -singlefile \
            -jpeg \
            -r "$PDF_DPI" \
            -jpegopt "quality=$PDF_JPEG_QUALITY" \
            "$pdf" \
            "$CURRENT_WORK/__page" \
            >/dev/null 2>&1
        then

            echo
            err "PDF第 $page 页转换失败"
            return 1
        fi

        if [ ! -f "$rendered" ]; then

            echo
            err "PDF第 $page 页没有生成 JPEG"
            err "临时目录：$CURRENT_WORK"

            ls -lah "$CURRENT_WORK" 2>/dev/null || true

            return 1
        fi

        # ====================================================
        # 处理 JPEG
        # ====================================================

        if ! process_image "$rendered"; then

            echo
            err "PDF第 $page 页图片处理失败"

            return 1
        fi

        rm -f "$rendered"

    done

    echo

    return 0
}

# ============================================================
# 获取图片列表
# ============================================================

get_images() {

    local dir="$1"

    find "$dir" \
        -maxdepth 1 \
        -type f \
        \( \
            -iname '*.jpg' \
            -o -iname '*.jpeg' \
            -o -iname '*.png' \
            -o -iname '*.gif' \
            -o -iname '*.webp' \
        \) \
        -printf '%f\n' |
        sort -V
}

# ============================================================
# 创建临时目录
# ============================================================

prepare_work() {

    local name="$1"

    CURRENT_WORK="$TMP_DIR/$name"

    mkdir -p "$CURRENT_WORK"

    # 防止重复运行留下旧文件
    rm -f "$CURRENT_WORK"/*.jpg

    IMAGE_INDEX=0
}

# ============================================================
# 从 PDF 文件名提取卷号
# ============================================================

get_volume() {

    local filename="$1"
    local number=""

    # Vol.01
    # Vol01
    # Volume01
    # Volume.01

    if [[ "$filename" =~ [Vv][Oo][Ll][Uu]?[Mm]?[Ee]?[\._\ -]*([0-9]+) ]]
    then

        number="${BASH_REMATCH[1]}"

    elif [[ "$filename" =~ ([0-9]+)[^0-9]*\.pdf$ ]]
    then

        number="${BASH_REMATCH[1]}"

    else

        echo "01"
        return
    fi

    printf "%02d" "$((10#$number))"
}

# ============================================================
# 判断 CBZ 是否已经完成
#
# 这里只检查：
#   1. 文件存在
#   2. ZIP 完整
#   3. ZIP 中有 JPG
#
# ============================================================

cbz_is_valid() {

    local cbz="$1"

    [ -f "$cbz" ] || return 1

    # ZIP 完整性
    unzip -tq "$cbz" >/dev/null 2>&1 || return 1

    # 至少有一张 JPG
    local count

    count=$(
        unzip -Z1 "$cbz" 2>/dev/null |
        grep -Ei '\.jpg$' |
        wc -l
    )

    [ "${count:-0}" -gt 0 ]
}

# ============================================================
# 创建 CBZ
# ============================================================

make_cbz() {

    local work="$1"
    local output="$2"

    local count

    count=$(find "$work" \
        -maxdepth 1 \
        -type f \
        -name '*.jpg' |
        wc -l)

    if [ "$count" -eq 0 ]; then
        err "没有找到输出图片"
        return 1
    fi

    info "生成 CBZ：$(basename "$output")"
    info "图片数量：$count"

    # ========================================================
    # 先删除同名旧文件
    # ========================================================

    rm -f "$output"

    # ========================================================
    # 创建 ZIP
    # ========================================================

    if ! (
        cd "$work" || exit 1

        zip \
            -q \
            -9 \
            "$output" \
            ./*.jpg
    )
    then

        err "CBZ 创建失败"
        err "保留临时目录：$work"

        return 1
    fi

    # ========================================================
    # 文件必须存在
    # ========================================================

    if [ ! -f "$output" ]; then

        err "zip 命令执行后没有找到 CBZ：$output"
        err "保留临时目录：$work"

        return 1
    fi

    # ========================================================
    # ZIP 完整性
    # ========================================================

    if ! unzip -tq "$output" >/dev/null 2>&1
    then

        err "CBZ ZIP 完整性检查失败"
        err "CBZ 已保留：$output"
        err "临时文件已保留：$work"

        return 1
    fi

    # ========================================================
    # 检查 JPG
    # ========================================================

    local zip_count

    zip_count=$(
        unzip -Z1 "$output" 2>/dev/null |
        grep -Ei '\.jpg$' |
        wc -l
    )

    if [ "${zip_count:-0}" -eq 0 ]; then

        err "CBZ 内没有 JPG"
        err "CBZ 已保留：$output"
        err "临时文件已保留：$work"

        return 1
    fi

    # ========================================================
    # 成功
    # ========================================================

    local size

    size="$(du -h "$output" | awk '{print $1}')"

    ok "完成：$(basename "$output")"
    echo "    文件大小：$size"
    echo "    图片数量：$zip_count"

    return 0
}

# ============================================================
# 处理 PDF 卷
# ============================================================

process_pdf_volume() {

    local pdf="$1"
    local output="$2"
    local work_name="$3"

    prepare_work "$work_name"

    if ! process_pdf "$pdf"
    then

        err "PDF处理失败：$(basename "$pdf")"
        err "临时文件保留：$CURRENT_WORK"

        return 1
    fi

    if ! make_cbz "$CURRENT_WORK" "$output"
    then

        err "CBZ创建失败：$(basename "$pdf")"
        return 1
    fi

    return 0
}

# ============================================================
# 处理图片目录
# ============================================================

process_image_directory() {

    local dir="$1"
    local output="$2"
    local work_name="$3"

    prepare_work "$work_name"

    local files=()

    while IFS= read -r file
    do
        files+=("$file")
    done < <(get_images "$dir")

    if [ "${#files[@]}" -eq 0 ]; then

        err "目录没有图片：$dir"
        return 1
    fi

    info "源图片：${#files[@]}"

    local i
    local file

    for ((i=0; i<${#files[@]}; i++))
    do

        file="${files[$i]}"

        printf "\r    图片 %d/%d" \
            "$((i + 1))" \
            "${#files[@]}"

        if ! process_image "$dir/$file"
        then

            echo
            err "图片处理失败：$file"

            return 1
        fi

    done

    echo

    make_cbz "$CURRENT_WORK" "$output"
}

# ============================================================
# 自动识别漫画名称
# ============================================================

detect_manga_name() {

    local pdf

    pdf=$(
        find "$BASE_DIR" \
            -maxdepth 1 \
            -type f \
            -iname '*.pdf' |
        sort -V |
        head -n 1
    )

    if [ -n "$pdf" ]; then

        local name

        name="$(basename "$pdf" .pdf)"

        # 删除：
        # .Vol.01
        # Vol.01
        # _Vol01
        # -Vol01

        name=$(
            echo "$name" |
            sed -E \
            's/[._ -]*[Vv]ol(ume)?[._ -]*[0-9]+.*$//'
        )

        if [ -n "$name" ]; then
            echo "$name"
        else
            basename "$BASE_DIR"
        fi

        return
    fi

    basename "$BASE_DIR"
}

# ============================================================
# 主程序
# ============================================================

main() {

    echo
    echo "============================================================"
    echo " Manga → CBZ Converter v${VERSION}"
    echo "============================================================"
    echo
    echo "工作目录："
    echo "  $BASE_DIR"
    echo

    check_dependencies

    mkdir -p "$OUTPUT_DIR"
    mkdir -p "$TMP_DIR"

    local manga_name

    manga_name="$(detect_manga_name)"

    info "漫画名称：$manga_name"
    info "输出目录：$OUTPUT_DIR"

    echo

    # ========================================================
    # PDF 模式
    # ========================================================

    local pdfs=()

    while IFS= read -r file
    do
        pdfs+=("$file")
    done < <(
        find "$BASE_DIR" \
            -maxdepth 1 \
            -type f \
            -iname '*.pdf' \
            -printf '%f\n' |
        sort -V
    )

    if [ "${#pdfs[@]}" -gt 0 ]; then

        echo "============================================================"
        echo "检测到 ${#pdfs[@]} 个 PDF 卷"
        echo "============================================================"
        echo

        local i
        local pdf
        local volume
        local output

        for ((i=0; i<${#pdfs[@]}; i++))
        do

            pdf="${pdfs[$i]}"

            volume="$(get_volume "$pdf")"

            output="$OUTPUT_DIR/${manga_name} Vol.${volume}.cbz"

            echo "============================================================"
            echo "处理 Vol.${volume}  [$((i + 1))/${#pdfs[@]}]"
            echo "============================================================"

            # ==================================================
            # 已经成功就跳过
            # ==================================================

            if cbz_is_valid "$output"
            then

                ok "已完成，跳过：$(basename "$output")"
                echo

                continue
            fi

            # ==================================================
            # 开始
            # ==================================================

            if process_pdf_volume \
                "$BASE_DIR/$pdf" \
                "$output" \
                "Vol_${volume}"
            then

                ok "Vol.${volume}完成"

            else

                err "Vol.${volume}失败"
                warn "临时文件保留在：$CURRENT_WORK"
                warn "继续处理下一卷"

            fi

            echo

        done

    else

        # ====================================================
        # 查找卷目录
        # ====================================================

        local dirs=()

        while IFS= read -r dir
        do
            dirs+=("$dir")
        done < <(
            find "$BASE_DIR" \
                -mindepth 1 \
                -maxdepth 1 \
                -type d \
                ! -name "cbz" \
                ! -name ".cbz_tmp" \
                -printf '%f\n' |
            sort -V
        )

        if [ "${#dirs[@]}" -gt 0 ]; then

            echo "============================================================"
            echo "检测到 ${#dirs[@]} 个卷目录"
            echo "============================================================"
            echo

            local i
            local dir
            local volume
            local output

            for ((i=0; i<${#dirs[@]}; i++))
            do

                dir="${dirs[$i]}"

                if [[ "$dir" =~ ([0-9]+)$ ]]
                then
                    volume=$(printf "%02d" "$((10#${BASH_REMATCH[1]}))")
                else
                    volume=$(printf "%02d" "$((i + 1))")
                fi

                output="$OUTPUT_DIR/${manga_name} Vol.${volume}.cbz"

                echo "============================================================"
                echo "处理 Vol.${volume}  [$((i + 1))/${#dirs[@]}]"
                echo "============================================================"

                if cbz_is_valid "$output"
                then

                    ok "已完成，跳过：$(basename "$output")"
                    echo

                    continue
                fi

                if process_image_directory \
                    "$BASE_DIR/$dir" \
                    "$output" \
                    "Vol_${volume}"
                then

                    ok "Vol.${volume}完成"

                else

                    err "Vol.${volume}失败"
                    warn "临时文件保留：$CURRENT_WORK"

                fi

                echo

            done

        else

            # =================================================
            # 当前目录直接图片
            # =================================================

            local images=()

            while IFS= read -r file
            do
                images+=("$file")
            done < <(get_images "$BASE_DIR")

            if [ "${#images[@]}" -gt 0 ]; then

                local output

                output="$OUTPUT_DIR/${manga_name}.cbz"

                echo "============================================================"
                echo "检测到当前目录直接存放图片"
                echo "============================================================"
                echo

                if cbz_is_valid "$output"
                then

                    ok "已完成，跳过：$(basename "$output")"

                else

                    process_image_directory \
                        "$BASE_DIR" \
                        "$output" \
                        "single"

                fi

            else

                err "当前目录没有找到 PDF、图片或卷目录。"
                exit 1

            fi

        fi

    fi

    echo
    echo "============================================================"
    echo "处理结束"
    echo "============================================================"
    echo
    echo "CBZ目录："
    echo "  $OUTPUT_DIR"
    echo

    if compgen -G "$OUTPUT_DIR/*.cbz" >/dev/null 2>&1
    then

        ls -lh "$OUTPUT_DIR"/*.cbz

    else

        warn "目前没有生成 CBZ。"

    fi

    echo
    echo "临时目录："
    echo "  $TMP_DIR"
    echo
}

main "$@"
```
