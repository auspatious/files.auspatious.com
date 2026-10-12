# files.auspatious.com

A single-file, dependency-free browser for the public S3 bucket behind
<https://files.auspatious.com>. One `index.html`: no build step, no framework.

- Folder navigation via the URL hash (`#some/folder/`), with breadcrumbs
- Prefix filter, sortable columns, S3 pagination
- Auspatious styling (Space Grotesk and Inclusive Sans from Google Fonts)

## How it works

The page calls the S3 `ListObjectsV2` REST API on the bucket
(`list-type=2`, `delimiter=/`) and renders folders and files. File links use the
public CloudFront domain. Settings are three constants at the top of the script:

| Constant | Purpose |
| --- | --- |
| `BUCKET` | S3 REST endpoint used for listing |
| `MASK` | Public base URL used for file links |
| `PAGE` | Items per page |

The bucket must allow anonymous `s3:ListBucket` and CORS `GET` from the site
origin. `index.html` itself is hidden from the listing.

## Develop

```sh
python3 -m http.server 8080   # then open http://127.0.0.1:8080/
```

Local testing lists the live bucket, so the bucket CORS rules need to allow
`http://127.0.0.1:8080` (or just test after deploying).

## Deploy

`index.html` lives at the root of the bucket, behind CloudFront.

```sh
aws s3 cp index.html s3://files.auspatious.com/index.html \
  --content-type "text/html; charset=utf-8" \
  --cache-control "no-cache"

aws cloudfront create-invalidation \
  --distribution-id E3OHG0DRR8V4YI --paths "/" "/index.html"
```

`no-cache` makes browsers revalidate on each load. The invalidation clears
the CloudFront copy; it usually completes within a minute or two. Check the
live file matches with `curl -s https://files.auspatious.com/ | md5` against
`md5 -q index.html`.

## Heritage

This started as a customised copy of
[qoomon/aws-s3-bucket-browser](https://github.com/qoomon/aws-s3-bucket-browser)
("S3 Bucket Explorer" v3.1.2, MIT licensed, Vue and Buefy based). The current
version is a rewrite in plain JavaScript with a new design and logo. It keeps the
original's approach (S3 list API, hash-based navigation, continuation-token paging)
but drops the markdown preview, iOS install links and zip download. Thanks to the
upstream authors.

## License

[Apache 2.0](LICENSE)
