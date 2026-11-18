# metabase-infra

Infrastructure for Supercast's self-hosted Metabase at `metabase.internal.supercast.com`.

## Image

`image/Dockerfile` runs the official Metabase jar on a JDK we pin ourselves, so JDK security fixes don't have to wait on Metabase shipping a new image.
Both inputs are pinned by tag and digest: `METABASE_IMAGE` (where the jar, certs and `run_metabase.sh` come from) and `JAVA_IMAGE` (the JRE).

Images live in ECR at `380046298144.dkr.ecr.us-west-2.amazonaws.com/metabase`, with immutable tags of the form `<metabase version>-<jdk distro><jdk version>`, e.g. `v0.65.5.1-corretto25.0.5.1.2`.

### Build and push

```sh
REG=380046298144.dkr.ecr.us-west-2.amazonaws.com
TAG=v0.65.5.1-corretto25.0.5.1.2

aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin $REG
docker build --platform linux/amd64 -t $REG/metabase:$TAG image/
docker push $REG/metabase:$TAG
```

### Verifying the running JDK

As an admin, open `https://metabase.internal.supercast.com/api/bug-reporting/details` and check `system-info.java.runtime.version`.

## Hosting

Currently a Cloud66 Docker stack.
Planned: ECS Fargate + RDS Postgres managed by Terraform in this repo, reusing the `supercast-terraform-state` S3 backend under the `metabase/` key.
