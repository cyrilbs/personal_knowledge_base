# Obsidian

## install 

### download app image
```
mv Obsidian-1.12.7.AppImage ~/Applications
ln -s ~/Applications/Obsidian-1.12.7.AppImage /usr/local/bin/obsidian
obsidian  --no-sandbox
```

### desk shortcut
```
nano ~/.local/share/applications/obsidian-nosandbox.desktop
  
[Desktop Entry]
Name=Obsidian (No Sandbox)
Comment=Launch Obsidian with --no-sandbox
Exec=/usr/local/bin/obsidian --no-sandbox
Icon=obsidian
Terminal=false
Type=Application
Categories=Office;

update-desktop-database ~/.local/share/applications/
```

## templates

### create one
```
cd /home/cyril/obsidian/yourname/templates

vi tech.md

create one hotkey to easily insert a template in a new note
```

### filter in graph view
![[Pasted image 20260327152848.png]]