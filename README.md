                 User Selects Folder / file
                         │
                         ▼
                Electron Renderer
                         │
               IPC Request (Scan Folder)
                         │
                         ▼
              Electron Main Process
                         │
          Recursively Scan Directory
                         │
                         ▼
           Return File Metadata (IPC)
                         │
          User Confirms Upload
                         │
               IPC Upload Request
                         │
                         ▼
               Upload Manager (Queue)
          (Fixed Concurrency Limit)
                         │
                         ▼
          Multipart Upload to Amazon S3
        (Split Large Files into Chunks)
                         │
             Parallel Chunk Uploads
                         │
                         ▼
        Aggregate Upload Progress (IPC)
                         │
                         ▼
          Renderer Updates Progress UI
                         │
                         ▼
      Every 10 Uploaded Files → AWS SQS
                         │
                         ▼
      Worker/Lambda Sends Email Notifications


### Design Decisions
- Electron renderer handles only UI; the main process manages file operations and uploads.
- IPC communication separates UI logic from system-level operations.
- Upload queue with a fixed concurrency limit prevents resource exhaustion.
- Large files use Amazon S3 multipart upload for better performance and reliability.
- Parallel chunk uploads improve upload throughput.
- Aggregate chunk-level progress into file-level progress for real-time UI updates.
- Persist the S3 UploadId and completed parts locally to support resumable uploads.
- On restart or network failure, resume uploading only the remaining chunks instead of restarting the entire file.
- Publish a message to AWS SQS after every 10 uploaded files to trigger asynchronous email notifications.
- Use asynchronous processing (SQS + Worker/Lambda) so notifications never block file uploads.
