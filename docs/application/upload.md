---
sidebar_position: 8
---

# Upload File

TinhTinh provide built-in `storage` middleware for update and store file.

TinhTinh default built-in struct `File` is:

```go
type File struct {
	FileName     string
	OriginalName string
	Encoding     string
	MimeType     string
	Size         int64
	Destination  string
	FieldName    string
	Path         string
}
```

TinhTinh are supported 3 types for upload:
| Name | Description |
|-|-|
| FileInterceptor | Upload one file |
| FilesInterceptor | Upload multiple file |
| FileFieldsInterceptor | Upload multiple file each multiple key |

## Basic Usage 

```go
import (
  "github.com/tinh-tinh/tinhtinh/v2/core"
  "github.com/tinh-tinh/tinhtinh/v2/middleware/storage"
)

store := &storage.DiskOptions{
  Destination: func(r *http.Request, file *multipart.FileHeader) string {
    return "./upload"
  },
  FileName: func(r *http.Request, file *multipart.FileHeader) string {
    uniqueSuffix := time.Now().Format("20060102150405") + "-" + file.Filename
    return uniqueSuffix
  },
}

ctrl.Use(core.FileInterceptor(storage.UploadFileOption{
  Storage: store,
})).Post("", func(ctx core.Ctx) error {

  return ctx.JSON(core.Map{
    "data": ctx.UploadedFile().OriginalName,
  })
})
```

If you need upload multiple files, you can use `FilesInterceptor`:

```go
ctrl.Use(core.FilesInterceptor(storage.UploadFileOption{
  Storage: store,
})).Post("", func(ctx core.Ctx) error {
  files := ctx.UploadedFiles()

  return ctx.JSON(core.Map{
    "data": len(files),
  })
})
```

Multiple files with multiple key from client

```go
ctrl.Use(
  core.FileFieldsInterceptor(storage.UploadFileOption{
    Storage: store,
  }, storage.FieldFile{
    Name:     "file1",
    MaxCount: 2,
  }, storage.FieldFile{
    Name:     "file2",
    MaxCount: 2,
  }),
).Post("", func(ctx core.Ctx) error {
  files := ctx.UploadedFieldFile()

  idx := 0
  for k, v := range files {
    for _, file := range v {
      fmt.Printf("Filed %s with file %v\n", k, file.FileName)
      idx++
    }
  }

  return ctx.JSON(core.Map{
    "data": idx,
  })
})
```
