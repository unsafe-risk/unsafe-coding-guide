# golang WASM vscode setting

## vscode setting

create `.vscode/setting.json`

```json
{
    "go.toolsEnvVars": {
        "GOOS": "js",
        "GOARCH": "wasm",
        "GOROOT": "/usr/local/go",
        "GOPATH": "/home/corey/src/goprojects"
    }
}
```

#go #wasm #vscode