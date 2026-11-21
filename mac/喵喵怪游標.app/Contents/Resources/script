#!/bin/bash
# 喵喵怪游標 for Mac — 一鍵安裝（Mousecape）
set -u

SELF_DIR="$(cd "$(dirname "$0")" 2>/dev/null && pwd)"
if [ -d "$SELF_DIR/capes" ]; then
  RES="$SELF_DIR"
else
  RES="$(cd "$SELF_DIR/../Resources" 2>/dev/null && pwd)"
fi
CAPES="$RES/capes"
SUPPORT="$HOME/Library/Application Support/Mousecape"
DEST="$SUPPORT/capes"

alert() {
  osascript - "$1" <<'APPLESCRIPT' >/dev/null 2>&1
on run argv
  display dialog (item 1 of argv) buttons {"好"} default button 1 with title "喵喵怪游標" with icon note
end run
APPLESCRIPT
}

choose2() {
  osascript - "$1" "$2" "$3" "$4" <<'APPLESCRIPT' 2>/dev/null
on run argv
  set r to display dialog (item 1 of argv) buttons {(item 2 of argv), (item 3 of argv)} default button ((item 4 of argv) as integer) with title "喵喵怪游標" with icon note
  return button returned of r
end run
APPLESCRIPT
}

find_mc() {
  if [ -d "/Applications/Mousecape.app" ]; then
    printf '%s\n' "/Applications/Mousecape.app"
    return 0
  fi
  if [ -d "$HOME/Applications/Mousecape.app" ]; then
    printf '%s\n' "$HOME/Applications/Mousecape.app"
    return 0
  fi
  return 1
}

mc_version() {
  /usr/bin/defaults read "$1/Contents/Info.plist" CFBundleVersion 2>/dev/null | tr -dc '0-9'
}

os_major() {
  sw_vers -productVersion 2>/dev/null | cut -d. -f1
}

pick_zip() {
  case "$(uname -m)" in
    arm64) printf '%s\n' "$RES/Mousecape-1820-arm64.zip" ;;
    *)     printf '%s\n' "$RES/Mousecape-0.0.6.zip" ;;
  esac
}

install_mc() {
  local dest="$1" zip="$2" tmp
  tmp="$(mktemp -d "${TMPDIR:-/tmp}/chesthair.XXXXXX")" || return 1
  if ! ditto -x -k "$zip" "$tmp" >/dev/null 2>&1 || [ ! -d "$tmp/Mousecape.app" ]; then
    rm -rf "$tmp"
    return 1
  fi
  if [ -e "$dest/Mousecape.app" ]; then
    osascript -e 'tell application "Mousecape" to quit' >/dev/null 2>&1
    sleep 1
    rm -rf "$dest/Mousecape.app" 2>/dev/null
  fi
  if ! cp -R "$tmp/Mousecape.app" "$dest/" 2>/dev/null; then
    rm -rf "$tmp"
    return 1
  fi
  rm -rf "$tmp"
  xattr -dr com.apple.quarantine "$dest/Mousecape.app" >/dev/null 2>&1
  chmod +x "$dest/Mousecape.app/Contents/MacOS/mousecloak" >/dev/null 2>&1
  printf '%s\n' "$dest/Mousecape.app"
}

install_fresh() {
  local zip mc
  zip="$(pick_zip)"
  if [ ! -f "$zip" ]; then
    return 1
  fi
  if mc="$(install_mc "/Applications" "$zip")"; then
    printf '%s\n' "$mc"
    return 0
  fi
  mkdir -p "$HOME/Applications" 2>/dev/null
  if mc="$(install_mc "$HOME/Applications" "$zip")"; then
    printf '%s\n' "$mc"
    return 0
  fi
  return 1
}

ensure_mc() {
  local mc ver major
  major="$(os_major)"
  if mc="$(find_mc)"; then
    ver="$(mc_version "$mc")"
    if [ "$(uname -m)" = "arm64" ] && [ "${major:-0}" -ge 26 ] && [ "${ver:-0}" -lt 1820 ]; then
      local btn
      btn="$(choose2 "偵測到舊版 Mousecape（版本 ${ver:-不明}）。
這個版本在 macOS ${major} 無法正常套用游標。

要換成相容版本嗎？" "取消" "更換" "2")"
      [ "$btn" = "更換" ] || return 2
      install_fresh
      return $?
    fi
    printf '%s\n' "$mc"
    return 0
  fi
  install_fresh
}

CHOICE="$(osascript <<'APPLESCRIPT' 2>/dev/null
set itemList to {"香蕉天使（喵喵怪游標）", "仙子休息居所（二代）", "萬聖節魔女喵喵", "扭扭喵喵", "仙女棒喵喵", "跑跑", "狗狗", "還原 macOS 預設游標", "移除喵喵怪游標"}
set picked to choose from list itemList with title "喵喵怪游標 for Mac" with prompt "請選擇要安裝的游標主題：" default items {"香蕉天使（喵喵怪游標）"} OK button name "繼續" cancel button name "取消"
if picked is false then
  return "__CANCEL__"
end if
return item 1 of picked
APPLESCRIPT
)"

[ -z "$CHOICE" ] && exit 0
[ "$CHOICE" = "__CANCEL__" ] && exit 0

KEY=""
NAME=""
ACTION="install"
case "$CHOICE" in
  "香蕉天使（喵喵怪游標）") KEY="banana"; NAME="$CHOICE" ;;
  "仙子休息居所（二代）") KEY="fairy"; NAME="$CHOICE" ;;
  "萬聖節魔女喵喵") KEY="halloween"; NAME="$CHOICE" ;;
  "扭扭喵喵") KEY="niuniu"; NAME="$CHOICE" ;;
  "仙女棒喵喵") KEY="wand"; NAME="$CHOICE" ;;
  "跑跑") KEY="run"; NAME="$CHOICE" ;;
  "狗狗") KEY="dog"; NAME="$CHOICE" ;;
  "還原 macOS 預設游標") ACTION="restore" ;;
  "移除喵喵怪游標") ACTION="remove" ;;
  *) exit 0 ;;
esac

case "$ACTION" in
  restore)
    if MC="$(find_mc)"; then
      "$MC/Contents/MacOS/mousecloak" -r --suppressCopyright >/dev/null 2>&1
      alert "已還原 macOS 預設游標。"
    else
      alert "目前沒有安裝 Mousecape，游標應該已經是系統預設值。"
    fi
    ;;

  remove)
    BTN="$(choose2 "確定要移除喵喵怪游標嗎？
會還原成 macOS 預設游標，並刪除已匯入的主題。" "取消" "移除" "1")"
    if [ "$BTN" = "移除" ]; then
      if MC="$(find_mc)"; then
        "$MC/Contents/MacOS/mousecloak" -r --suppressCopyright >/dev/null 2>&1
      fi
      rm -f "$DEST/banana.cape" "$DEST/fairy.cape" "$DEST/halloween.cape" \
            "$DEST/niuniu.cape" "$DEST/wand.cape" "$DEST/run.cape" \
            "$DEST/dog.cape" 2>/dev/null
      alert "已移除喵喵怪游標，並還原成 macOS 預設游標。"
    fi
    ;;

  install)
    if [ ! -f "$CAPES/$KEY.cape" ]; then
      alert "找不到主題檔案：$KEY.cape"
      exit 1
    fi
    MC="$(ensure_mc)"
    RC=$?
    if [ "$RC" -ne 0 ]; then
      [ "$RC" -eq 2 ] && exit 0
      alert "無法安裝 Mousecape。
請確認網路連線正常，或手動到 https://mousecape.app 下載安裝後再試一次。"
      exit 1
    fi
    MCLOAK="$MC/Contents/MacOS/mousecloak"
    [ -x "$MCLOAK" ] || chmod +x "$MCLOAK" 2>/dev/null
    if [ ! -x "$MCLOAK" ]; then
      alert "Mousecape 安裝不完整（找不到 mousecloak）。
請先開啟 Mousecape 一次，再回來試。"
      exit 1
    fi

    mkdir -p "$DEST" 2>/dev/null
    cp "$CAPES/$KEY.cape" "$DEST/$KEY.cape" 2>/dev/null

    OUT="$("$MCLOAK" -a "$DEST/$KEY.cape" --suppressCopyright 2>&1)"
    ESC="$(printf '\033')"
    CLEAN="$(printf '%s' "$OUT" | sed "s/${ESC}\[[0-9;]*m//g" 2>/dev/null)"
    if printf '%s' "$CLEAN" | grep -q "successfully"; then
      BTN="$(choose2 "已套用「$NAME」！

游標會立即改變。
若希望重開機後仍保留，請在 Mousecape 按「Install Helper Tool」。

要現在開啟 Mousecape 嗎？" "好" "開啟 Mousecape" "1")"
      if [ "$BTN" = "開啟 Mousecape" ]; then
        open "$MC" >/dev/null 2>&1
      fi
    else
      alert "套用失敗。

$CLEAN"
    fi
    ;;
esac

exit 0
