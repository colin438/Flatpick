# Flatpick

### A small script To select a flatpak via the General name instead of the App ID for example If you run app sober it launches sober, the normal way would be running org.vinegarhq.Sober. Flatpak names are also inconsistent On how that are names so this script also solves that.

## Install

#### Edit your ~/.zshrc via nano  ~/.zshrc  (or any other text editor, nano is better though)
#### go to the end of the file and add the code below and after you paste it <div style="border: 2px solid #000; padding: 10px; background-color: #f0f0f0; margin-top: 10px;">
  <code>source ~/.zshrc</code>
</div>     
<div style="border: 1px solid #ccc; padding: 10px; border-radius: 5px;">

```bash
# Universal Flatpak-only app launcher with fuzzy matching
app() {
    local query="$1"
    shift

    if [[ -z "$query" ]]; then
        echo "⚠️ Usage: app <name>"
        return 1
    fi

    echo "🔍 Searching Flatpak apps matching: '$query'..."

    # List of user-friendly names and their corresponding Flatpak IDs
    declare -A app_map=(
        [sober]="org.vinegarhq.Sober"
        [firefox]="org.mozilla.firefox"
        [gimp]="org.gimp.GIMP"
    )

    # Check if user-friendly name exists in the map
    if [[ -n "${app_map[$query]}" ]]; then
        echo "✅ Running Flatpak: ${app_map[$query]}"
        flatpak run "${app_map[$query]}" "$@"
        return 0
    fi

    # If not, perform fuzzy matching across all installed apps
    local matches=()
    while IFS= read -r line; do
        matches+=("$line")
    done < <(flatpak list --app --columns=application)

    # If matches found, allow user to choose one
    local count=${#matches[@]}
    if (( count == 0 )); then
        echo "❌ No Flatpak app found matching '$query'"
        return 1
    elif (( count == 1 )); then
        echo "✅ Running Flatpak: ${matches[0]}"
        flatpak run "${matches[0]}" "$@"
    else
        echo "🚨 Multiple matches for '$query':"
        for i in "${!matches[@]}"; do
            echo "[$((i+1))] ${matches[$i]}"
        done
        read -rp "Choose app number: " choice
        if [[ "$choice" =~ ^[0-9]+$ ]] && (( choice >= 1 && choice <= count )); then
            local selected="${matches[$((choice - 1))]}"
            echo "✅ Running Flatpak: $selected"
            flatpak run "$selected" "$@"
        else
            echo "❌ Invalid choice"
            return 1
        fi
    fi
}

#### To apply the script update your zsh by running source ~/.zshrc      
