檔案管理
===============

在進行檔案管理時，首要措施是要確保使用者無法向任何動態功能提供資料。在某些與言，例如 PHP，將資料傳給動態函式是一個嚴重的安全隱憂。Go 是一個編譯語言，這代表沒有 `include` 函式，同時函式無法被動態加載。

File uploads should only be restricted to authenticated users. After guaranteeing that file uploads are only made by authenticated users, another important aspect of security is to make sure that only accepted filetypes can be uploaded to the server (_whitelisting_). This check can be made using the following Go function that detects MIME types: `func DetectContentType(data []byte) string`

A simple program that reads a file and identifies its MIME type is attached. The most relevant parts are the following:

```go
{...}
// Write our file to a buffer
// Why 512 bytes? See http://golang.org/pkg/net/http/#DetectContentType
buff := make([]byte, 512)

_, err = file.Read(buff)
{...}
//Result - Our detected filetype
filetype := http.DetectContentType(buff)
```

After identifying the filetype, an additional step is required to validate the filetype against a whitelist of allowed filetypes. In the example, this is achieved in the following section:

```go
{...}
switch filetype {
case "image/jpeg", "image/jpg":
  fmt.Println(filetype)
case "image/gif":
  fmt.Println(filetype)
case "image/png":
  fmt.Println(filetype)
default:
  fmt.Println("unknown file type uploaded")
}
{...}
```

Files uploaded by users should not be stored in the web context of the application. Instead, files should be stored in a content server or in a database. An important note is for the selected file upload destination not to have execution privileges.

If the file server that hosts user uploads is \*NIX based, make sure to implement safety mechanisms like chrooted environment or mounting the target file directory as a logical drive.

Again, since Go is a compiled language, the usual risk of uploading files that contain malicious code that can be interpreted on the server-side is non-existent.

In the case of dynamic redirects, user data should not be passed. If it is required by your application, additional steps must be taken to keep the application safe. These checks include accepting only properly validated data and relative path URLs.

Additionally, when passing data into dynamic redirects, it is important to make sure that directory and file paths are mapped to indexes of pre-defined lists of paths and to use said indexes.

Never send the absolute file path to the user, always use relative paths.

Set the server permissions regarding the application files and resources to `read-only`, and when a file is uploaded, scan the file for viruses and malware.
