# Binary Distrubtion API

바이너리의 배포를 돕는 간단한 API 입니다.

## Purpose

개발을 하는 과정에서 우리는 수 많은 바이너리를 지속적으로 배포해야 합니다. 그리고 그 바이너리들을 도움을 줄 수 있는 사람들에게 쉽게 전달할 수 있도록 배포해야 합니다.

이 프로젝트는 그것을 위한 간단한 기반을 제공합니다. AWS Serverless 를 사용하여 낮은 가격으로 이것들을 제공할 수 있도록 합니다.

## Features

다음의 기능을 제공합니다. 자세한 API 상세는 Usage 를 참고하세요.

- 새로운 바이너리를 업로드
- 업로드한 바이너리를 제거
- 업로드한 바이너리를 버전 별로 조회

## API Listing

다음의 API 들로 구성되어 있습니다.

### X-Auth-Token

보안 헤더로 이 값이 올바른 요청에 대해서만 서버가 처리를 진행합니다. 이는 최소한의 보안장치로 다른 사람이 함부로 이 API 를 사용하는 것을 막습니다. 이는 `config-api`를 통해 미리 발급된 Token 을 다음과 같이 HTTP Header 로 전달하면 됩니다.

```bash
curl -H "X-Auth-Token: YOUR-SECRET-TOKEN"
```

### createDistribution

새로운 Distribution 을 업로드하기 위한 URL 을 받습니다.

```
PUT /{serviceName}/{platform}/{version}
```

예를 들어 다음과 같이 curl 로 요청할 수 있습니다.

```bash
curl -XPUT "https://your-api-server/hello-service/android/hello-20181125.apk" -H "X-Auth-Token: YOUR-SECRET-TOKEN"
```

이 반환 값으로 Upload 하기 위한 S3 의 Signed URL 이 반환됩니다. 이를 이용하여 curl 로 바로 업로드를 진행하려면 다음과 같이 `"` 문자를 제거한 후에 파일 업로드를 진행하면 됩니다.

```bash
curl -T your-binary-file "$( \
  curl -XPUT \
	  "https://your-api-server/hello-service/android/hello-20181125.apk" \
	  -H "X-Auth-Token: YOUR-SECRET-TOKEN" \
  | tr -d '"')"
```

### deleteDistribution

예전에 업로드한 Distribution 을 제거합니다.

```
DELETE /{serviceName}/{platform}/{version}
```

예를 들어 다음과 같이 curl 로 요청할 수 있습니다.

```bash
curl -XDELETE "https://your-api-server/hello-service/android/hello-20181125.apk"
```

### listAllDistributions

업로드한 모든 플랫폼 별 최신 버전을 확인합니다.

```
GET /{serviceName}
```

예를 들어 다음과 같이 curl 로 요청할 수 있습니다.

```
curl -XGET https://your-api-server/hello-service

# response
{
	"service": "hello-service",
	"platforms": {
		"linux": ["https://your-download-server/hello-service/android/hello-20181125.apk"]
	}
}
```

### listPlatformDistributions

특정 플랫폼에 소속된 버전 목록을 확인합니다.

```
GET /{serviceName}
```

예를 들어 다음과 같이 curl 로 요청할 수 있습니다.

```
curl -XGET https://your-api-service/hello-service/android

# response
{
	"service": "hello-service",
	"platform": "android",
	"versions": ["https://your-download-server/hello-world/android/hello-20181125.apk"]
}
```

## Provisioned Service

이 프로젝트는 `yyt.life` 소속으로, 당신이 만약 이 서비스에 참여하고 있다면 이미 제공되는 다음 서비스 도메인을 사용할 수 있습니다.

- API 서버로 다음 주소를 사용하고: `https://api.yyt.life/d/`
- 다음 주소를 다운로드 주소로 사용할 수 있습니다: `https://d.yyt.life/`

물론 그 전에 `yyt.life` 운영자에게 요청하여 `X-Auth-Token`으로 사용할 token 을 미리 발급받아야 합니다.

### CI

다음의 API call 을 CI job 하단부에 넣어주면 됩니다.

```bash
curl -T new-binary-version "$( \
  curl -XPUT \
	  "https://api.yyt.life/d/your-service/your-platform/new-binary-version.apk" \
	  -H "X-Auth-Token: YOUR-SECRET-TOKEN" \
  | tr -d '"')"
```

만약 Travis-CI 와 같은 공개될 수 있는 CI 를 사용한다면 Secret Token 은 안전을 위해 secret 이나 적어도 environment variable 등으로 넣어주어야 합니다.
