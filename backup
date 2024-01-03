#!/usr/bin/env bash

backup_dir="$HOME/Accessory/Extern/Backup"

[[ ! -d "$src" ]] && echo "[ERROR] Given directory '$src' does not exist" && exit 1
[[ ! -d "$backup_dir" ]] && echo "[ERROR] Backup directory '$backup_dir' does not exist" && exit 1

destination="$backup_dir/$(basename "$src")""---""$(date '+%Y-%m-%d---%H-%M-%S')"
file_list="$destination"".txt"

latest_files="$destination""---date.txt"
biggest_files="$destination""---size.txt"

cd "$src" || exit

file_paths=()
while IFS= read -r -d '' file; do

    excluded=false
    for substring in "${excluded_subpaths[@]}"; do
        if [[ "$file" == *"$substring"* ]]; then
            excluded=true
            break
        fi
    done

    included=true
    for substring in "${included_subpaths[@]}"; do
        if [[ "$file" != *"$substring"* ]]; then
            included=false
            break
        fi
    done

    if [[ $excluded == false && $included == true ]] ; then
        file_paths+=("${file#"$src/"}")
    fi

done < <(find "$src" -type f -print0)

printf '%s\n' "${file_paths[@]}" > "$file_list"

7z a "$destination"".7z" -ir@"$file_list"

# for long file lists we skip the creation of info files
if (( $(stat -f%z "$file_list") > 500000 )); then
    [[ -f "$file_list" ]] && rm "$file_list"
    exit 0
fi

echo "Sorting by file size..."
while read -r file_path; do
    file_size=$(stat -f "%z" "$src/$file_path")
    echo "$file_size $file_path"
done < "$file_list" | sort -rn | head -n 100 > "$biggest_files"

echo "Sorting by modified date..."
while read -r file_path; do
    if [[ "$file_path" == *"/."* ]]; then continue; fi # We skip invisible files
    modified_date=$(stat -f "%Sm" -t "%Y-%m-%d %H:%M:%S" "$src/$file_path")
    echo "$modified_date $file_path"
done < "$file_list" | sort -r | head -n 100 > "$latest_files"

[[ -f "$file_list" ]] && rm "$file_list"