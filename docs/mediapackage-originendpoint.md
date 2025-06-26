# MediaPackage OriginEndpoint

The MediaPackage OriginEndpoint resource allows you to create AWS Elemental MediaPackage origin endpoints for packaging and delivering live video content.

## Overview

AWS Elemental MediaPackage origin endpoints are used to package live video content from MediaPackage channels into various streaming formats. Origin endpoints support multiple packaging formats including HLS, DASH, CMAF, and Microsoft Smooth Streaming, with advanced features like encryption, ad insertion, and content protection.

## API Reference

### Resource Types

- **Composite Resource**: `XOriginEndpoint` (cluster-scoped)
- **Claim**: `OriginEndpoint` (namespace-scoped)
- **API Version**: `mediapackage.aws.livewyer.io/v1alpha1`

### Specification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `channelId` | string | Yes | The ID of the Channel the OriginEndpoint is associated with |
| `description` | string | No | A short text description of the OriginEndpoint |
| `manifestName` | string | No | A short string appended to the end of the OriginEndpoint URL |
| `origination` | string | No | Control whether the OriginEndpoint is used for just-in-time packaging (`ALLOW`, `DENY`) |
| `startoverWindowSeconds` | integer | No | Maximum duration (seconds) of content to retain for startover playback |
| `timeDelaySeconds` | integer | No | Amount of delay (seconds) to enforce on the playback of live content |
| `whitelist` | array | No | List of source IP addresses or CIDR blocks allowed to access the OriginEndpoint |
| `authorization` | object | No | Parameters for CDN authorization |
| `cmafPackage` | object | No | Common Media Application Format (CMAF) packaging configuration |
| `dashPackage` | object | No | Dynamic Adaptive Streaming over HTTP (DASH) packaging configuration |
| `hlsPackage` | object | No | HTTP Live Streaming (HLS) packaging configuration |
| `mssPackage` | object | No | Microsoft Smooth Streaming (MSS) packaging configuration |
| `region` | string | No | AWS region where the origin endpoint will be created |
| `tags` | object | No | Key-value pairs to associate with the origin endpoint |

### Authorization Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `CdnIdentifierSecret` | string | Yes | ARN for the secret in Secrets Manager that your CDN uses for authorization |
| `SecretsRoleArn` | string | Yes | ARN for the IAM role that allows MediaPackage to communicate with AWS Secrets Manager |

### CMAF Package Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Encryption` | object | No | CMAF encryption configuration |
| `HlsManifests` | array | No | List of HLS manifest configurations |
| `SegmentDurationSeconds` | integer | No | Duration (in seconds) of each segment |
| `SegmentPrefix` | string | No | Optional custom string prepended to each segment name |
| `StreamSelection` | object | No | Stream selection configuration |

### DASH Package Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `AdsOnDeliveryRestrictions` | string | No | Flags on SCTE-35 segmentation descriptors for ad markers |
| `AdTriggers` | array | No | SCTE-35 message types treated as ad markers |
| `Encryption` | object | No | DASH encryption configuration |
| `IncludeIframeOnlyStream` | boolean | No | When enabled, includes an I-Frame only stream |
| `ManifestLayout` | string | No | Position of tags in the MPD (`FULL`, `COMPACT`) |
| `ManifestWindowSeconds` | integer | No | Time window (in seconds) contained in each manifest |
| `MinBufferTimeSeconds` | integer | No | Minimum duration for player to buffer before playback |
| `MinUpdatePeriodSeconds` | integer | No | Minimum duration between potential MPD changes |
| `PeriodTriggers` | array | No | List of triggers for MPD partitioning |
| `Profile` | string | No | DASH profile type (`NONE`, `HBBTV_1_5`, `HYBRIDCAST`, `DVB_DASH_2014`) |
| `SegmentDurationSeconds` | integer | No | Duration (in seconds) of each segment |
| `SegmentTemplateFormat` | string | No | Type of SegmentTemplate included in MPD |
| `StreamSelection` | object | No | Stream selection configuration |
| `SuggestedPresentationDelaySeconds` | integer | No | Duration to delay live content before presentation |
| `UtcTiming` | string | No | Type of UTCTiming included in MPD |
| `UtcTimingUri` | string | No | Value attribute of UTCTiming field |

### HLS Package Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `AdMarkers` | string | No | How ad markers are included (`NONE`, `SCTE35_ENHANCED`, `PASSTHROUGH`, `DATERANGE`) |
| `AdsOnDeliveryRestrictions` | string | No | Flags on SCTE-35 segmentation descriptors for ad markers |
| `AdTriggers` | array | No | SCTE-35 message types treated as ad markers |
| `Encryption` | object | No | HLS encryption configuration |
| `IncludeDvbSubtitles` | boolean | No | Pass through DVB subtitles into output |
| `IncludeIframeOnlyStream` | boolean | No | Include I-Frame only stream in output |
| `PlaylistType` | string | No | HLS playlist type (`NONE`, `EVENT`, `VOD`) |
| `PlaylistWindowSeconds` | integer | No | Total duration (in seconds) of each playlist |
| `ProgramDateTimeIntervalSeconds` | integer | No | Interval between EXT-X-PROGRAM-DATE-TIME tags |
| `SegmentDurationSeconds` | integer | No | Duration (in seconds) of each segment (1-30) |
| `StreamSelection` | object | No | Stream selection configuration |
| `UseAudioRenditionGroup` | boolean | No | Place audio streams in rendition groups |

### MSS Package Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Encryption` | object | No | MSS encryption configuration |
| `ManifestWindowSeconds` | integer | No | Time window (in seconds) contained in each manifest |
| `SegmentDurationSeconds` | integer | No | Duration (in seconds) of each segment |
| `StreamSelection` | object | No | Stream selection configuration |

## Examples

### Basic HLS Origin Endpoint

```yaml
apiVersion: mediapackage.aws.livewyer.io/v1alpha1
kind: OriginEndpoint
metadata:
  name: basic-hls-endpoint
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    channelId: my-channel
    description: "Basic HLS endpoint for live streaming"
    manifestName: index
    hlsPackage:
      AdMarkers: SCTE35_ENHANCED
      PlaylistType: EVENT
      PlaylistWindowSeconds: 60
      SegmentDurationSeconds: 6
      ProgramDateTimeIntervalSeconds: 0
    tags:
      Environment: production
      Format: hls
```

### DASH Origin Endpoint with Encryption

```yaml
apiVersion: mediapackage.aws.livewyer.io/v1alpha1
kind: OriginEndpoint
metadata:
  name: dash-encrypted-endpoint
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    channelId: secure-channel
    description: "DASH endpoint with DRM encryption"
    startoverWindowSeconds: 300
    timeDelaySeconds: 30
    dashPackage:
      Profile: HBBTV_1_5
      SegmentDurationSeconds: 6
      ManifestLayout: FULL
      MinBufferTimeSeconds: 4
      MinUpdatePeriodSeconds: 2
      SuggestedPresentationDelaySeconds: 25
      Encryption:
        SpekeKeyProvider:
          SystemIds:
            - "81376844-f976-481e-a84e-cc25d39b0b33"
          Url: "https://drm-proxy.example.com/proxy"
          ResourceId: "secure-content-id"
          RoleArn: "arn:aws:iam::123456789012:role/MediaPackageSpekeRole"
    whitelist:
      - "203.0.113.0/24"
      - "198.51.100.0/24"
    tags:
      Environment: production
      Format: dash
      Security: encrypted
```

### CMAF Origin Endpoint

```yaml
apiVersion: mediapackage.aws.livewyer.io/v1alpha1
kind: OriginEndpoint
metadata:
  name: cmaf-endpoint
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    channelId: live-channel
    description: "CMAF endpoint for multi-format delivery"
    cmafPackage:
      SegmentDurationSeconds: 4
      SegmentPrefix: "live_"
      HlsManifests:
        - Id: "primary"
          ManifestName: "primary"
          PlaylistType: EVENT
          PlaylistWindowSeconds: 60
          ProgramDateTimeIntervalSeconds: 0
          AdMarkers: SCTE35_ENHANCED
        - Id: "backup"
          ManifestName: "backup"
          PlaylistType: EVENT
          PlaylistWindowSeconds: 60
      StreamSelection:
        MaxVideoBitsPerSecond: 8000000
        MinVideoBitsPerSecond: 1000000
        StreamOrder: VIDEO_BITRATE_ASCENDING
    tags:
      Environment: production
      Format: cmaf
      Redundancy: enabled
```

### Origin Endpoint with CDN Authorization

```yaml
apiVersion: mediapackage.aws.livewyer.io/v1alpha1
kind: OriginEndpoint
metadata:
  name: cdn-authorized-endpoint
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    channelId: premium-channel
    description: "HLS endpoint with CDN authorization"
    authorization:
      CdnIdentifierSecret: "arn:aws:secretsmanager:us-east-1:123456789012:secret:cdn-auth-secret"
      SecretsRoleArn: "arn:aws:iam::123456789012:role/MediaPackageSecretsRole"
    hlsPackage:
      AdMarkers: PASSTHROUGH
      PlaylistType: EVENT
      PlaylistWindowSeconds: 300
      SegmentDurationSeconds: 10
      UseAudioRenditionGroup: true
      IncludeIframeOnlyStream: true
    tags:
      Environment: production
      CDN: authorized
      Content: premium
```

### Microsoft Smooth Streaming Endpoint

```yaml
apiVersion: mediapackage.aws.livewyer.io/v1alpha1
kind: OriginEndpoint
metadata:
  name: mss-endpoint
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    channelId: legacy-channel
    description: "Microsoft Smooth Streaming for legacy compatibility"
    mssPackage:
      ManifestWindowSeconds: 60
      SegmentDurationSeconds: 10
      StreamSelection:
        MaxVideoBitsPerSecond: 5000000
        MinVideoBitsPerSecond: 500000
        StreamOrder: VIDEO_BITRATE_DESCENDING
    tags:
      Environment: production
      Format: mss
      Legacy: silverlight
```

### Multi-Format Origin Endpoint

```yaml
apiVersion: mediapackage.aws.livewyer.io/v1alpha1
kind: OriginEndpoint
metadata:
  name: multi-format-endpoint
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    channelId: versatile-channel
    description: "Multi-format endpoint supporting HLS and DASH"
    origination: ALLOW
    startoverWindowSeconds: 86400
    whitelist:
      - "0.0.0.0/0"
    hlsPackage:
      AdMarkers: DATERANGE
      PlaylistType: EVENT
      PlaylistWindowSeconds: 300
      SegmentDurationSeconds: 6
      ProgramDateTimeIntervalSeconds: 60
      IncludeIframeOnlyStream: true
      UseAudioRenditionGroup: true
    tags:
      Environment: production
      Accessibility: public
      Features: multi-format
```

## Status

The origin endpoint status includes information about the AWS resource state.

```yaml
status:
  conditions:
    - type: Ready
      status: "True"
      reason: Available
  atProvider:
    # AWS-specific resource information
    properties: {}
```

## Packaging Formats

### HLS (HTTP Live Streaming)
- **Best for**: iOS devices, Safari browsers, general streaming
- **Features**: Adaptive bitrate, DVB subtitles, I-frame playlists
- **File Extension**: `.m3u8` (manifest), `.ts` (segments)

### DASH (Dynamic Adaptive Streaming over HTTP)
- **Best for**: Cross-platform streaming, MPEG-DASH players
- **Features**: Adaptive bitrate, multiple codecs, flexible manifests
- **File Extension**: `.mpd` (manifest), `.mp4` (segments)

### CMAF (Common Media Application Format)
- **Best for**: Multi-format delivery, reduced storage
- **Features**: Single encode for multiple formats, HLS and DASH support
- **File Extension**: `.m4s` (segments)

### MSS (Microsoft Smooth Streaming)
- **Best for**: Legacy Microsoft/Silverlight applications
- **Features**: Adaptive bitrate, Windows ecosystem integration
- **File Extension**: `.ism` (manifest), `.ismv` (segments)

## Security Features

### DRM Integration
- **SPEKE**: Secure Packager and Encoder Key Exchange
- **Multi-DRM**: Support for multiple DRM systems
- **Key Rotation**: Automatic encryption key rotation

### Access Control
- **IP Whitelisting**: Restrict access by IP address/CIDR blocks
- **CDN Authorization**: Integrate with CDN authentication
- **Secrets Management**: Secure key storage in AWS Secrets Manager

## Use Cases

### Live Streaming
Package live video content for immediate delivery to viewers across multiple devices and platforms.

### Time-Shifted Viewing
Enable startover and catch-up functionality with configurable time windows.

### Multi-Platform Delivery
Serve content to various devices using appropriate packaging formats.

### Premium Content
Implement DRM protection and access controls for premium content delivery.

## AWS Documentation

For more detailed information about MediaPackage origin endpoints, refer to the official AWS documentation:

- [AWS MediaPackage Origin Endpoints](https://docs.aws.amazon.com/mediapackage/latest/ug/endpoints.html)
- [AWS Cloud Control API - MediaPackage OriginEndpoint](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-operations.html)
- [AWS::MediaPackage::OriginEndpoint CloudFormation Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-mediapackage-originendpoint.html)

## Notes

- **Channel Dependency**: The MediaPackage channel must exist before creating origin endpoints
- **Format Selection**: Choose packaging format based on target device compatibility
- **Security**: Configure appropriate access controls and encryption for your use case
- **Performance**: Consider segment duration and playlist window for optimal performance
- **CDN Integration**: Use CDN authorization for production deployments
- **Monitoring**: Monitor endpoint performance and adjust configurations as needed
- **Startover**: Configure startover window based on your time-shift requirements
- **Ad Insertion**: Configure SCTE-35 ad markers for server-side ad insertion (SSAI)
