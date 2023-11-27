**Common Response for all the post methods**

```typescript
// common-response.decorator.ts
import { SetMetadata } from "@nestjs/common";

export const CommonResponse = (responseType: string) =>
  SetMetadata("commonResponse", { responseType });

// common-response.interceptor.ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";

@Injectable()
export class CommonResponseInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const commonResponse = Reflect.getMetadata(
      "commonResponse",
      context.getHandler()
    );

    return next.handle().pipe(
      map((data) => ({
        code: 200,
        status: "success",
        type: commonResponse.responseType,
        url: data.url || null,
        data:
          data.data ||
          `This is the webhook response you intend to display on the chat window`,
      }))
    );
  }
}

// main.module.ts
import { Module } from "@nestjs/common";
import { APP_INTERCEPTOR } from "@nestjs/core";
import { CommonResponseInterceptor } from "./common-response.interceptor";

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: CommonResponseInterceptor,
    },
  ],
})
export class MainModule {}
```

Now, your service can return an object with `url` and `data` properties, and the interceptor will use them accordingly:

```typescript
// example.service.ts
import { Injectable } from "@nestjs/common";

@Injectable()
export class ExampleService {
  getTextResponse(): { url?: string; data: string } {
    return { data: "Text response data" };
  }

  getImageResponse(): { url?: string; data: string } {
    return { url: "http://example.com/image-url", data: "Image response data" };
  }

  getDocumentResponse(): { url?: string; data: string } {
    return {
      url: "http://example.com/document-url",
      data: "Document response data",
    };
  }

  getVideoResponse(): { url?: string; data: string } {
    return { url: "http://example.com/video-url", data: "Video response data" };
  }
}
```

And your controller methods will remain similar:

```typescript
// example.controller.ts
import { Controller, Post } from "@nestjs/common";
import { CommonResponse } from "./common-response.decorator";
import { ExampleService } from "./example.service";

@Controller("example")
export class ExampleController {
  constructor(private readonly exampleService: ExampleService) {}

  @Post("text")
  @CommonResponse("text")
  postText(): { url?: string; data: string } {
    return this.exampleService.getTextResponse();
  }

  @Post("image")
  @CommonResponse("image")
  postImage(): { url?: string; data: string } {
    return this.exampleService.getImageResponse();
  }

  @Post("document")
  @CommonResponse("document")
  postDocument(): { url?: string; data: string } {
    return this.exampleService.getDocumentResponse();
  }

  @Post("video")
  @CommonResponse("video")
  postVideo(): { url?: string; data: string } {
    return this.exampleService.getVideoResponse();
  }
}
```

This way, the `CommonResponseInterceptor` will dynamically use the `url` and `data` properties returned by your service.
