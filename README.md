# FFmpeg Audio Merging on Windows (PowerShell)

This guide shows how to properly merge multiple `.m4a` audio files on Windows using PowerShell and FFmpeg.

## Method: Safe Merge with Re-encoding

```powershell
# Create a temporary directory for processed files
New-Item -ItemType Directory -Path "processed" -Force

# Process each file individually to ensure compatibility
Get-ChildItem -Filter "*.m4a" | Sort-Object Name | ForEach-Object {
    ffmpeg -i $_.FullName -c:a aac -strict experimental "processed\$($_.Name)"
}

# Create merge list
$lines = Get-ChildItem -Path "processed" -Filter "*.m4a" | Sort-Object Name | ForEach-Object { 
    "file '$($_.FullName)'" 
}
[System.IO.File]::WriteAllLines("file_list.txt", $lines, (New-Object System.Text.UTF8Encoding($false)))

# Merge all files
ffmpeg -f concat -safe 0 -i file_list.txt -c copy output.m4a
```

## Key Features:

1. **Pre-processing**:
   - Creates a clean `processed` directory
   - Re-encodes each file to standardized AAC format

2. **Safe Merging**:
   - Uses proper UTF-8 encoding without BOM
   - Includes `-safe 0` for path handling
   - Maintains original quality with `-c copy`

3. **Sorting**:
   - Files are processed in alphabetical order

## Troubleshooting:

- If you get "moov atom not found" errors, try:
  ```powershell
  ffmpeg -i problematic.m4a -c:a aac -strict experimental fixed.m4a
  ```
- For permission issues, run PowerShell as Administrator
