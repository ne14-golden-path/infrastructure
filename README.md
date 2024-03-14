# infrastructure
Shared infrastructure

# build docker

```bash
# portal.web image (VS Code)
docker build . -t ne1410s/portal.web:0.0.1 --push

# portal.api image (VS)
docker build . -t ne1410s/portal.api:0.0.1 -f ".\ne14.portal.api\Dockerfile" --force-rm --secret id=nuget_config_file,src="C:\temp\nuget-docker.golden-path.config" --push

# services.docs image (VS)
docker build . -t ne1410s/services.docs:0.0.1 -f ".\ne14.services.docs.app\Dockerfile" --force-rm --secret id=nuget_config_file,src="C:\temp\nuget-docker.golden-path.config" --push
```