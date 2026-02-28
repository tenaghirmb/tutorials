# VS Code Configuration

## User Settings(JSON)

```json
{
    // Automatically format and import packages when saving (standard in GoLand)
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.organizeImports": "always"
    },
    // Turn on inline prompts (parameter name prompts, one of GoLand’s strongest features)
    "go.inlayHints.parameterNames": true,
    "go.inlayHints.assignVariableTypes": true,
    "go.inlayHints.compositeLiteralFields": true,
    "go.inlayHints.functionReturnNames": true,
    "go.inlayHints.constantValues": true,
    
    // Automatically complete brackets and quotes
    "editor.autoClosingBrackets": "always",
    "editor.autoClosingQuotes": "always"
}
```

## Local Development + Remote Cluster

Debugging Kubebuilder projects locally

**Step 1**: Synchronize CRD Execute the following command in your local terminal (ensure KUBECONFIG is configured `/etc/rancher/k3s/k3s.yaml`)
```bash
make install
```
> This will install your custom resource definition (CRD) directly into the remote Host's K3s.

**Step 2**: Configure VS Code launch.json Creating a local debug configuration is crucial for telling the program where to find the Kubernetes cluster

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Controller Locally",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${workspaceFolder}/main.go",
            "args": [
                "--leader-elect=false", // Local debugging must turn off the host selection, otherwise it will cause slow startup due to delays.
                "--zap-devel"           // Enable detailed logs to facilitate viewing the Reconcile process
            ],
            "env": {
                // Force point to remote cluster configuration file
                "KUBECONFIG": "${userHome}/.kube/config" 
            },
            "showLog": true
        }
    ]
}
```

**Step 3**: Set a breakpoint anywhere in controllers/xxx_controller.go, then press F5 to start.

**Step 4**: In another terminal, use `kubectl apply -f config/samples/...` to create a resource and check if the breakpoints are triggered correctly.

> Tools: gopls, dlv, staticcheck `export GOPROXY=https://goproxy.cn,direct`

```bash
# install staticcheck (code lint)
go install honnef.co/go/tools/cmd/staticcheck@latest

# install dlv (debugging)
go install github.com/go-delve/delve/cmd/dlv@latest

# install gopls (Language server, providing code completion)
go install golang.org/x/tools/gopls@latest
```
