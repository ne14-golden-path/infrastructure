# infrastructure
Shared infrastructure

# build docker

```bash
# portal.web image (VS Code)
docker build . -t ne1410s/portal.web:0.0.1 --push

# portal.api image (VS)
docker build -f ".\ne14.portal.api\Dockerfile" --force-rm --tag portal.api --secret id=nuget_config_file,src="C:\temp\nuget-docker.golden-path.config" .
docker images
docker tag <IMAGE_ID> ne1410s/portal.api:<VERSION>
docker push ne1410s/portal.api:<VERSION>

# services.docs image (VS)
docker build -f ".\ne14.services.docs.app\Dockerfile" --force-rm --tag services.docs --secret id=nuget_config_file,src="C:\temp\nuget-docker.golden-path.config" .
docker images
docker tag <IMAGE_ID> ne1410s/services.docs:<VERSION>
docker push ne1410s/services.docs:<VERSION>
```