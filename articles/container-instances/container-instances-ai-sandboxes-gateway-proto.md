---
title: Sandbox gateway protobuf reference for serverless sandboxes (Preview)
description: Complete sandboxgateway.proto definition and C# CLI walkthrough for generating a gRPC client to interact with serverless sandboxes in Azure Container Instances.
ms.topic: reference
ms.author: dpeshwani
author: drishtipeshwani09
ms.service: azure-container-instances
services: container-instances
ms.date: 11/04/2026
ms.update-cycle: 1095-days
# Customer intent: As a developer of AI agents, I want the complete sandboxgateway.proto definition so that I can generate a gRPC client in my preferred language to interact with running serverless sandboxes in Azure Container Instances.
---

# Sandbox gateway protobuf reference for serverless sandboxes (preview)

This article provides the [`sandboxgateway.proto` definition](#protobuf-definition-sandboxgatewayproto) for the [sandbox gateway API](container-instances-ai-sandboxes-api-reference.md#sandbox-gateway-api) (`aci.sandboxgateway.SandboxGatewayService`) of [serverless sandboxes in Azure Container Instances](container-instances-ai-sandboxes.md) and shows how to [build a C# CLI tool](#build-a-c-cli-from-sandboxgatewayproto) from it to interact with a running sandbox.

To connect to a running sandbox with a gRPC client:

* **Get the sandbox connection string:** Call `GET {endpoint}/sandboxes/{sandbox}`. The response includes `properties.connectionString` in the format `address=<gateway-host>;token=<gateway-token>;token_type=bearer`.
* **Connect to the gateway endpoint:** Open a TLS gRPC connection to `https://<gateway-host>:443` using the `address` from the connection string.
* **Authenticate and route requests with the token:** Include `authorization: Bearer <gateway-token>` in the gRPC request headers (metadata). The gateway uses this token both to authorize the request and to identify the target sandbox.

## Build a C# CLI from sandboxgateway.proto

You can use the standard .NET gRPC tooling (`Grpc.Net.Client`, `Google.Protobuf`, and `Grpc.Tools`) to compile `sandboxgateway.proto` into a typed C# client and wrap it in a cross-platform command-line tool that runs on Linux, macOS, and Windows.

### 1. Create the project

Install the [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) on your Linux, macOS, or Windows machine, and create a new .NET 8 console application:

```bash
dotnet new console -n SandboxGatewayCli --framework net8.0
cd SandboxGatewayCli
```

### 2. Add sandboxgateway.proto to the project

1. Copy the [`sandboxgateway.proto` definition](#protobuf-definition-sandboxgatewayproto) into the `SandboxGatewayCli` folder as `sandboxgateway.proto`.
1. Replace the contents of `SandboxGatewayCli.csproj` with the following project file. It adds the gRPC packages and a `<Protobuf>` item, so that MSBuild generates the `SandboxGatewayService.SandboxGatewayServiceClient` class at build time:

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">

      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net8.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
      </PropertyGroup>

      <ItemGroup>
        <PackageReference Include="Google.Protobuf" Version="3.28.3" />
        <PackageReference Include="Grpc.Net.Client" Version="2.66.0" />
        <PackageReference Include="Grpc.Tools" Version="2.67.0" PrivateAssets="All" />
      </ItemGroup>

      <ItemGroup>
        <Protobuf Include="sandboxgateway.proto" GrpcServices="Client" />
      </ItemGroup>

    </Project>
    ```

### 3. Implement the CLI in Program.cs

Replace `Program.cs` with the following sample implementation. Pass the sandbox's `properties.connectionString` (`address=<gateway-host>;token=<gateway-token>;token_type=bearer`) to each command with `--connectionstring` or `-cs`.

While `sandboxgateway.proto` defines a full set of operations (including interactive shells, port forwarding, directory management, file search/watch, and container execution—see the [Sandbox gateway API reference](container-instances-ai-sandboxes-api-reference.md#sandbox-gateway-api)), this sample CLI implements a few common commands (`wait`, `exec`, `logs`, `write-file`, `read-file`, and `wait-op`) to demonstrate how to call the generated client, which you can extend with any other RPCs from the protobuf definition. Use `wait-op` with the operation `endpoint` (`<operation-gateway-host>`) and `accessToken` (`<operation-token>`) that the `saveSnapshot` and `resize` actions return, in the form `address=<operation-gateway-host>;token=<operation-token>;token_type=bearer`:

```csharp
using System.Text;
using ACI.SandboxGateway.Api.V1;
using Google.Protobuf;
using Grpc.Core;
using Grpc.Net.Client;

var (connectionString, cmdArgs) = ParseConnectionStringOption(args);
if (string.IsNullOrWhiteSpace(connectionString) || cmdArgs.Length == 0)
{
    Console.Error.WriteLine(
        "Usage: dotnet run -- -cs \"<connection-string>\" <wait | exec <cmd> [args...] | logs [tail] | write-file <path> <text> | read-file <path> | wait-op [timeout-seconds]>");
    return 1;
}

if (!TryParseConnectionString(connectionString, out var host, out var token))
{
    Console.Error.WriteLine("The connection string must have the form 'address=<gateway-host>;token=<gateway-token>;token_type=bearer'. A stopped sandbox has no connection string.");
    return 1;
}

using var channel = GrpcChannel.ForAddress($"https://{host}:443");
var client = new SandboxGatewayService.SandboxGatewayServiceClient(channel);
var headers = new Metadata { { "authorization", $"Bearer {token}" } };

switch (cmdArgs[0])
{
    case "wait":
        var waitResp = await client.WaitForReadyAsync(
            new WaitForReadyRequest { TimeoutSeconds = 120 }, headers);
        Console.WriteLine($"Ready: {waitResp.Ready} (state: {waitResp.CurrentState})");
        return waitResp.Ready ? 0 : 1;

    case "exec":
        if (cmdArgs.Length < 2)
        {
            Console.Error.WriteLine("Usage: dotnet run -- -cs \"<connection-string>\" exec <command> [args...]");
            return 1;
        }
        var execReq = new ExecSandboxRequest
        {
            Command = cmdArgs[1],
            TimeoutSeconds = 60,
        };
        execReq.Args.AddRange(cmdArgs.Skip(2));
        var execResp = await client.ExecSandboxAsync(execReq, headers);
        if (!string.IsNullOrEmpty(execResp.Stdout)) Console.Write(execResp.Stdout);
        if (!string.IsNullOrEmpty(execResp.Stderr)) Console.Error.Write(execResp.Stderr);
        return execResp.ExitCode;

    case "logs":
        uint tail = cmdArgs.Length > 1 && uint.TryParse(cmdArgs[1], out var n) ? n : 50;
        var logsResp = await client.GetLogsAsync(
            new GetLogsRequest { Tail = tail, Timestamps = true }, headers);
        foreach (var entry in logsResp.Entries)
        {
            Console.WriteLine($"{entry.Timestamp} [{entry.Level}] {entry.Message}");
        }
        return 0;

    case "write-file":
        if (cmdArgs.Length < 3)
        {
            Console.Error.WriteLine("Usage: dotnet run -- -cs \"<connection-string>\" write-file <guest-path> <content>");
            return 1;
        }
        var writeResp = await client.WriteFileAsync(new WriteFileRequest
        {
            Path = cmdArgs[1],
            Content = ByteString.CopyFromUtf8(cmdArgs[2]),
        }, headers);
        Console.WriteLine($"Wrote {writeResp.BytesWritten} bytes to {cmdArgs[1]}");
        return 0;

    case "read-file":
        if (cmdArgs.Length < 2)
        {
            Console.Error.WriteLine("Usage: dotnet run -- -cs \"<connection-string>\" read-file <guest-path>");
            return 1;
        }
        using (var call = client.ReadFile(new ReadFileRequest { Path = cmdArgs[1] }, headers))
        {
            await foreach (var chunk in call.ResponseStream.ReadAllAsync())
            {
                Console.Write(Encoding.UTF8.GetString(chunk.Content.Span));
            }
        }
        return 0;

    case "wait-op":
        var opResp = await client.WaitForOperationCompleteAsync(new WaitForOperationCompleteRequest
        {
            TimeoutSeconds = cmdArgs.Length > 1 && long.TryParse(cmdArgs[1], out var t) ? t : 300,
        }, headers);
        Console.WriteLine($"Complete: {opResp.Complete} (state: {opResp.TerminalState}) {opResp.Reason}");
        return opResp.Complete && opResp.TerminalState == "succeeded" ? 0 : 1;

    default:
        Console.Error.WriteLine($"Unknown command: {cmdArgs[0]}");
        return 1;
}

static (string? ConnectionString, string[] RemainingArgs) ParseConnectionStringOption(string[] args)
{
    string? cs = null;
    var remaining = new List<string>();
    for (int i = 0; i < args.Length; i++)
    {
        if ((args[i].Equals("--connectionstring", StringComparison.OrdinalIgnoreCase) ||
             args[i].Equals("--connection-string", StringComparison.OrdinalIgnoreCase) ||
             args[i].Equals("-cs", StringComparison.OrdinalIgnoreCase)) &&
            i + 1 < args.Length)
        {
            cs = args[++i];
        }
        else
        {
            remaining.Add(args[i]);
        }
    }
    return (cs, remaining.ToArray());
}

static bool TryParseConnectionString(string connectionString, out string host, out string token)
{
    var parts = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
    foreach (var segment in connectionString.Split(';', StringSplitOptions.RemoveEmptyEntries))
    {
        var idx = segment.IndexOf('=');
        if (idx > 0)
        {
            parts[segment[..idx].Trim()] = segment[(idx + 1)..].Trim();
        }
    }

    host = string.Empty;
    token = string.Empty;
    if (!parts.TryGetValue("address", out var address) || !parts.TryGetValue("token", out var value) ||
        string.IsNullOrWhiteSpace(address) || string.IsNullOrWhiteSpace(value))
    {
        return false;
    }

    host = address.Replace("https://", "", StringComparison.OrdinalIgnoreCase)
                  .Replace("http://", "", StringComparison.OrdinalIgnoreCase)
                  .TrimEnd('/');
    var colon = host.LastIndexOf(':');
    if (colon > 0) host = host[..colon];
    token = value;
    return true;
}
```

### 4. Try out the CLI

1. Build the project:

    ```bash
    dotnet build
    ```

1. Run commands against the sandbox by passing `properties.connectionString` with `--connectionstring` or `-cs`:

    ```bash
    # Wait for the sandbox guest to be ready
    dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" wait

    # Execute a command inside the sandbox
    dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" exec /bin/sh -c "cat /etc/os-release; nproc"

    # Write and read a file in the sandbox
    dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" write-file /tmp/hello.txt $'hello from my computer\n'
    dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" read-file /tmp/hello.txt

    # Fetch the last 20 sandbox log entries
    dotnet run -- -cs "address=<gateway-host>;token=<gateway-token>;token_type=bearer" logs 20

    # Wait for a snapshot or resize operation, using the endpoint and accessToken that the action returns
    dotnet run -- -cs "address=<operation-gateway-host>;token=<operation-token>;token_type=bearer" wait-op 300
    ```

## Protobuf definition (sandboxgateway.proto)

Save the following protobuf definition as `sandboxgateway.proto`:

```protobuf
syntax = "proto3";

option csharp_namespace = "ACI.SandboxGateway.Api.V1";

package aci.sandboxgateway;

service SandboxGatewayService {
  // Run a one-shot command directly inside a running sandbox.
  rpc ExecSandbox(ExecSandboxRequest) returns (ExecSandboxResponse);

  // Wait for a sandbox to reach the ready (Running) state.
  rpc WaitForReady(WaitForReadyRequest) returns (WaitForReadyResponse);

  // Get a sandbox's basic definition and current state.
  rpc GetSandbox(GetSandboxRequest) returns (GetSandboxResponse);

  // Pause a running sandbox.
  rpc PauseSandbox(PauseSandboxRequest) returns (PauseSandboxResponse);

  // Resume a paused sandbox.
  rpc ResumeSandbox(ResumeSandboxRequest) returns (ResumeSandboxResponse);

  // Open an interactive or streaming shell session inside a running sandbox.
  rpc InteractiveShell(stream InteractiveShellRequest) returns (stream InteractiveShellResponse);

  // Interactively exec a command inside a running container.
  rpc ExecContainer(stream ExecContainerRequest) returns (stream ExecContainerResponse);

  // Forward a TCP connection to a port inside a running sandbox.
  rpc PortForward(stream PortForwardRequest) returns (stream PortForwardResponse);

  // Write raw bytes to a file path inside a running sandbox.
  rpc WriteFile(WriteFileRequest) returns (WriteFileResponse);

  // Stream raw bytes to a file path without buffering the full file.
  rpc WriteFileStream(stream WriteFileStreamRequest) returns (WriteFileResponse);

  // Fetch an Azure Blob directly into the sandbox using a SAS URL.
  rpc DownloadFromBlobstore(DownloadFromBlobstoreRequest) returns (FileTransferResponse);

  // Upload a sandbox file directly to Azure Blob Storage using a SAS URL.
  rpc DownloadFileToUrl(DownloadFileToUrlRequest) returns (FileTransferResponse);

  // Create a directory (and its parent tree) inside a running sandbox.
  rpc CreateDirectory(CreateDirectoryRequest) returns (CreateDirectoryResponse);

  // Bind-mount an existing file or directory onto a target path inside a running sandbox.
  rpc BindMount(BindMountRequest) returns (BindMountResponse);

  // Connect a running sandbox's guest to an iSCSI target.
  rpc ConnectIscsi(ConnectIscsiRequest) returns (ConnectIscsiResponse);

  // Read a file from a running sandbox, streamed in chunks.
  rpc ReadFile(ReadFileRequest) returns (stream ReadFileResponse);

  // Delete a file or directory inside a running sandbox.
  rpc DeleteFile(DeleteFileRequest) returns (DeleteFileResponse);

  // List the entries of a directory inside a running sandbox.
  rpc ListDirectory(ListDirectoryRequest) returns (ListDirectoryResponse);

  // Move or rename a file or directory inside a running sandbox.
  rpc Move(MoveRequest) returns (MoveResponse);

  // Search file contents under a path inside a running sandbox.
  rpc SearchContent(SearchContentRequest) returns (SearchContentResponse);

  // Replace occurrences of a pattern within a file in a running sandbox.
  rpc ReplaceContent(ReplaceContentRequest) returns (ReplaceContentResponse);

  // Watch a path inside a running sandbox for filesystem changes.
  rpc WatchChanges(WatchChangesRequest) returns (stream WatchChangesResponse);

  // Wait for a long-running operation (such as snapshot or resize) to complete.
  rpc WaitForOperationComplete(WaitForOperationCompleteRequest) returns (WaitForOperationCompleteResponse);

  // Read a running sandbox's log buffer.
  rpc GetLogs(GetLogsRequest) returns (GetLogsResponse);
}

message GetLogsRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  uint32 tail = 3;
  string level = 4;
  bool timestamps = 5;
  string cursor = 6;
}

message SandboxLogEntry {
  string timestamp = 1;
  string level = 2;
  string message = 3;
}

message GetLogsResponse {
  repeated SandboxLogEntry entries = 1;
  string next_cursor = 2;
  bool gap_detected = 3;
  bool has_more = 4;
}

message WriteFileRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  string path = 3;
  bytes content = 4;
  uint32 mode = 5;
  bool append = 6;
  string container_id = 7;
}

message WriteFileResponse {
  int64 bytes_written = 1;
}

message DownloadFromBlobstoreRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  string source_sas_url = 3;
  string path = 4;
  uint32 mode = 5;
  string container_id = 6;
}

message DownloadFileToUrlRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  string path = 3;
  string destination_sas_url = 4;
  string container_id = 5;
}

message FileTransferResponse {
  int64 bytes_transferred = 1;
}

message WriteFileStreamRequest {
  oneof request {
    WriteFileStreamStart start = 1;
    bytes content = 2;
  }
}

message WriteFileStreamStart {
  string path = 1;
  uint32 mode = 2;
  bool append = 3;
  string container_id = 4;
}

message CreateDirectoryRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  string path = 3;
  uint32 mode = 4;
  string container_id = 5;
}

message CreateDirectoryResponse {
}

message BindMountRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  string source = 3;
  string target = 4;
  bool readonly = 5;
  bool recursive = 6;
}

message BindMountResponse {
}

message ReadFileRequest {
  string path = 1;
  int64 offset = 2;
  int32 max_chunk_size = 3;
  string container_id = 4;
}

message ReadFileResponse {
  bytes content = 1;
  int64 next_offset = 2;
  bool eof = 3;
}

message DeleteFileRequest {
  string path = 1;
  bool recursive = 2;
  string container_id = 3;
}

message DeleteFileResponse {
}

message ListDirectoryRequest {
  string path = 1;
  string container_id = 2;
}

message DirectoryEntry {
  string name = 1;
  bool is_directory = 2;
  bool is_symlink = 3;
  int64 size = 4;
  int64 modified_time_ticks = 5;
  uint32 mode = 6;
}

message ListDirectoryResponse {
  repeated DirectoryEntry entries = 1;
}

message MoveRequest {
  string source = 1;
  string destination = 2;
  bool overwrite = 3;
  string container_id = 4;
}

message MoveResponse {
}

message SearchContentRequest {
  string path = 1;
  string pattern = 2;
  bool is_regex = 3;
  bool case_sensitive = 4;
  int32 max_results = 5;
  string container_id = 6;
}

message SearchContentMatch {
  string path = 1;
  int64 line_number = 2;
  string line = 3;
}

message SearchContentResponse {
  repeated SearchContentMatch matches = 1;
  bool truncated = 2;
}

message ReplaceContentRequest {
  string path = 1;
  string pattern = 2;
  string replacement = 3;
  bool is_regex = 4;
  bool case_sensitive = 5;
  string container_id = 6;
}

message ReplaceContentResponse {
  int64 replacements = 1;
}

message WatchChangesRequest {
  string path = 1;
  bool recursive = 2;
}

enum WatchChangeType {
  WATCH_CHANGE_UNKNOWN = 0;
  WATCH_CHANGE_CREATED = 1;
  WATCH_CHANGE_MODIFIED = 2;
  WATCH_CHANGE_DELETED = 3;
  WATCH_CHANGE_RENAMED = 4;
}

message WatchChangesResponse {
  WatchChangeType change_type = 1;
  string path = 2;
  string old_path = 3;
  int64 timestamp_ticks = 4;
}

message WaitForReadyRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  int64 timeout_seconds = 3;
}

message WaitForReadyResponse {
  bool ready = 1;
  string current_state = 2;
  string reason = 3;
  string error_code = 4;
  int64 startup_duration_ms = 5;
  int64 vm_start_duration_ms = 6;
  int64 bootstrap_duration_ms = 7;
}

message WaitForOperationCompleteRequest {
  int64 timeout_seconds = 1;
}

message WaitForOperationCompleteResponse {
  bool complete = 1;
  string terminal_state = 2;
  string reason = 3;
  string error_code = 4;
}

message GetSandboxRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
}

message SandboxDefinition {
  int32 cores = 1;
  int32 memory_in_gb = 2;
  int32 disk_in_gb = 3;
  string operating_system = 4;
  string image = 5;
  string target_state = 6;
  string current_state = 7;
  string failure_message = 8;
  int64 startup_duration_ms = 9;
  int64 vm_start_duration_ms = 10;
  int64 bootstrap_duration_ms = 11;
}

message GetSandboxResponse {
  SandboxDefinition sandbox = 1;
}

message PauseSandboxRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
}

message PauseSandboxResponse {
  string current_state = 1;
}

message ResumeSandboxRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
}

message ResumeSandboxResponse {
  string current_state = 1;
}

message ExecSandboxRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  string command = 3;
  repeated string args = 4;
  string working_directory = 5;
  map<string, string> environment = 6;
  int32 timeout_seconds = 7;
  string user = 8;
}

message ExecSandboxResponse {
  int32 exit_code = 1;
  string stdout = 2;
  string stderr = 3;
  bool timed_out = 4;
}

message InteractiveShellRequest {
  oneof request {
    StartInteractiveShellRequest start = 1;
    ShellInputData input = 2;
    WindowSizeChange window_size = 3;
    SignalRequest signal = 4;
  }
}

message StartInteractiveShellRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  string shell = 3;
  map<string, string> environment = 4;
  string working_directory = 5;
  int32 initial_rows = 6;
  int32 initial_cols = 7;
  repeated string args = 8;
  string user = 9;
  bool non_terminal = 10;
}

message ShellInputData {
  bytes data = 1;
  bool eof = 2;
}

message WindowSizeChange {
  int32 rows = 1;
  int32 cols = 2;
}

message SignalRequest {
  string signal = 1;
}

message InteractiveShellResponse {
  oneof response {
    ShellStartedResponse started = 1;
    ShellOutputData output = 2;
    ShellExitedResponse exited = 3;
    ShellErrorResponse error = 4;
  }
}

message ShellStartedResponse {
  int32 pid = 1;
}

message ShellOutputData {
  enum StreamType {
    STDOUT = 0;
    STDERR = 1;
  }
  StreamType stream = 1;
  bytes data = 2;
}

message ShellExitedResponse {
  int32 exit_code = 1;
}

message ShellErrorResponse {
  string message = 1;
}

message ExecContainerRequest {
  oneof request {
    StartExecContainerRequest start = 1;
    ExecInputData input = 2;
    ExecWindowSizeChange window_size = 3;
    ExecSignalRequest signal = 4;
  }
}

message StartExecContainerRequest {
  reserved 1, 2;
  reserved "namespace", "sandbox";
  string container_id = 3;
  repeated string cmd = 4;
  bool tty = 5;
  bool stdin = 6;
  string working_directory = 7;
  map<string, string> environment = 8;
  int32 initial_rows = 9;
  int32 initial_cols = 10;
}

message ExecInputData {
  bytes data = 1;
}

message ExecWindowSizeChange {
  int32 rows = 1;
  int32 cols = 2;
}

message ExecSignalRequest {
  string signal = 1;
}

message ExecContainerResponse {
  oneof response {
    ExecStartedResponse started = 1;
    ExecOutputData output = 2;
    ExecExitedResponse exited = 3;
    ExecErrorResponse error = 4;
  }
}

message ExecStartedResponse {
  int32 pid = 1;
}

message ExecOutputData {
  enum StreamType {
    STDOUT = 0;
    STDERR = 1;
  }
  StreamType stream = 1;
  bytes data = 2;
}

message ExecExitedResponse {
  int32 exit_code = 1;
}

message ExecErrorResponse {
  string message = 1;
}

message PortForwardRequest {
  oneof request {
    StartPortForwardRequest start = 1;
    PortForwardData data = 2;
    PortForwardClose close = 3;
  }
}

message StartPortForwardRequest {
  int32 port = 1;
  string host = 2;
}

message PortForwardData {
  bytes data = 1;
}

message PortForwardClose {
}

message PortForwardResponse {
  oneof response {
    PortForwardReady ready = 1;
    PortForwardData data = 2;
    PortForwardClosed closed = 3;
    PortForwardError error = 4;
  }
}

message PortForwardReady {
}

message PortForwardClosed {
}

message PortForwardError {
  string message = 1;
}

message ConnectIscsiRequest {
  string target_iqn = 1;
  string portal = 2;
  uint32 port = 3;
  uint32 sessions = 4;
  uint32 timeout_seconds = 5;
  bool format_if_empty = 6;
}

message ConnectIscsiResponse {
  repeated string session_ids = 1;
  repeated string devices = 2;
  string portal_address = 3;
  repeated string multipath_devices = 4;
}
```

## Related content

* [Serverless sandboxes REST and gateway API reference](container-instances-ai-sandboxes-api-reference.md)
* [Quickstart: Create your first serverless sandbox](container-instances-ai-sandboxes-quickstart.md)
* [Serverless sandboxes in Azure Container Instances](container-instances-ai-sandboxes.md)
