The `MediaManager` is an interface used for uploading media files and retrieving their URLs. Here are the main components:

### MediaManager Interface

```typescript
// src/domain/ports/out/infra/MediaManager.ts
export interface MediaFile {
  buffer: Buffer;
}

export interface MediaManager {
  uploadMedia(file: MediaFile, folder?: string): Promise<UploadMediaResult>;
  getMediaUrl(keyOrObjectId: string): Promise<string>;
}
```

### AwsS3MediaManager Class

The `AwsS3MediaManager` class implements the `MediaManager` interface. It handles media files using AWS S3.

```typescript
import { GetObjectCommand, PutObjectCommand, S3Client } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import { Constant, Injectable } from "@tsed/di";
import { $log } from "@tsed/logger";

import { MEDIA_MANAGER } from "../../../../constants/Token.js";
import type { MediaFile, MediaManager, UploadMediaResult } from "../../../../domain/ports/out/infra/MediaManager.js";

@Injectable({ token: MEDIA_MANAGER })
export class AwsS3MediaManager implements MediaManager {
  private client: S3Client;
  private readonly maxAttempts: number = 3;

  @Constant("envs.AWS_S3_BUCKET")
  private bucket!: string;

  @Constant("envs.AWS_REGION")
  private region!: string;

  @Constant("envs.AWS_ACCESS_KEY_ID")
  private accessKeyId!: string;

  @Constant("envs.AWS_SECRET_ACCESS_KEY")
  private accessKey!: string;

  @Constant("envs.AWS_S3_PREFIX")
  private keyPrefix?: string;

  @Constant("envs.MEDIA_URL_TTL_SECONDS")
  private urlExpiresInSeconds?: number;

  @Constant("envs.MEDIA_COMMON_FOLDER")
  private commonFolder?: string;

  getPlatform(): "AWS-S3" {
    return "AWS-S3";
  }

  private canConnect(): boolean {
    return Boolean(this.bucket && this.region && this.accessKeyId && this.accessKey);
  }

  private getClient(): S3Client {
    if (!this.client) {
      if (!this.canConnect()) {
        $log.error("media config missing: region/credentials/bucket");
        throw new Error("Media storage misconfigured");
      }
      this.client = new S3Client({
        region: this.region,
        credentials: { accessKeyId: this.accessKeyId, secretAccessKey: this.accessKey },
        maxAttempts: this.maxAttempts
      });
      $log.info("media client ready");
    }
    return this.client;
  }

  private sanitizedFileName(filename: string, folder?: string): string {
    const sanitized = filename.replace(/[^\w.\-]+/g, "_");
    const basePrefix = this.keyPrefix ? this.keyPrefix.replace(/^\/+|\/+$/g, "") : "";
    const folderPrefix = folder ? folder.replace(/^\/+|\/+$/g, "") : "";
    const combinedPrefix = [basePrefix, folderPrefix].filter(Boolean).join("/");
    const prefix = combinedPrefix ? combinedPrefix + "/" : "";
    const timestamp = Date.now();
    const random = Math.random().toString(36).slice(2, 10);
    return `${prefix}${timestamp}-${random}-${sanitized}`;
  }

  async uploadMedia(file: MediaFile, folder?: string): Promise<UploadMediaResult> {
    try {
      const client = this.getClient();
      const folderPath = folder ?? this.commonFolder;
      const key = this.sanitizedFileName(file.filename, folderPath);
      const command = new PutObjectCommand({
        Bucket: this.bucket,
        Key: key,
        Body: file.buffer,
        ContentType: file.mimeType
      });
      const response = await client.send(command);
      const etagRaw = response.ETag ?? "";
      const objectId = etagRaw.replace(/(^")|("$)/g, "");

      $log.info(`upload success key=${key}`);
      return {
        success: Boolean(objectId),
        relativePath: key,
        objectId
      };
    } catch (error) {
      $log.error(`upload error: ${error.message}`);
      throw error;
    }
  }

  async getMediaUrl(keyOrObjectId: string): Promise<string> {
    try {
      const client = this.getClient();
      const key = keyOrObjectId;
      const command = new GetObjectCommand({
        Bucket: this.bucket,
        Key: key
      });
      const expires = Number(this.urlExpiresInSeconds ?? 900);
      const url = await getSignedUrl(client, command, { expiresIn: expires });
      $log.info(`signed url created key=${key} expires=${expires}s`);
      return url;
    } catch (error) {
      $log.error(`signed url error: ${error.message}`);
      throw error;
    }
  }
}

```

### Constants

`MEDIA_MANAGER` is a unique symbol used for the `AwsS3MediaManager` class.

```typescript
// src/constants/Token.ts
export const MEDIA_MANAGER: unique symbol = Symbol.for("MediaManager");
```

### Providers

The `AwsS3MediaManager` class is provided using the `MEDIA_MANAGER` token.

```typescript
// src/adapter/providers.ts
import { AwsS3MediaManager } from "./out/infra/aws/AwsMediaManager.js";

const infraProviders: object[] = [
  { provide: MEDIA_MANAGER, useClass: AwsS3MediaManager },
  // ... existing code ...
];
```

This documentation provides a comprehensive overview of the `MediaManager` interface, its implementations, the token used, and the provider setup.
