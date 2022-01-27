## Developer Notes

Bumping version (e.g. to 1.21.6). Get latest _mainline_ release version from
<http://nginx.org/en/download.html>.

```
export OLD_VERSION=1.21.3
export VERSION=1.21.6
  # Download & verify nginx
pushd ~/Downloads
curl -OL http://nginx.org/keys/nginx_signing.key
curl -OL http://nginx.org/keys/mdounin.key
curl -OL http://nginx.org/download/nginx-$VERSION.tar.gz
curl -OL http://nginx.org/download/nginx-$VERSION.tar.gz.asc
mkdir /tmp/$$
gpg2 --homedir /tmp/$$ --import nginx_signing.key # The canonical signing key
gpg2 --homedir /tmp/$$ --import mdounin.key # Sometimes Maxim Dounin signs with his own key
gpg2 --homedir /tmp/$$ --verify nginx-$VERSION.tar.gz.asc
popd
  # Prepare the BOSH release
pushd ~/workspace/nginx-release
git pull -r --autostash
find packages/nginx -type f -print0 | \
  xargs -0 perl -pi -e \
  "s/nginx-${OLD_VERSION}/nginx-${VERSION}/g"
sed -i '' "s/$OLD_VERSION/$VERSION/g" README.md
bosh add-blob \
  ~/Downloads/nginx-${VERSION}.tar.gz \
  nginx/nginx-${VERSION}.tar.gz
vim config/blobs.yml
  # delete `nginx/nginx-${OLD_VERSION}.tar.gz` stanza
bosh create-release --force
  # authenticate against BOSH Lite (BOSH on VirtualBox)
export BOSH_ENVIRONMENT=vbox
export BOSH_CLIENT=admin
export BOSH_CLIENT_SECRET=`bosh int ~/deployments/vbox/creds.yml --path /admin_password`
  # upload & test the release
bosh upload-release
bosh -n -d nginx \
  deploy manifests/nginx-lite.yml --recreate
 # `bosh -e vbox vms`; browse to nginx VM
bosh -d nginx ssh
curl -I localhost # check for `HTTP/1.1 200 OK`
exit
bosh upload-blobs
bosh create-release \
  --final \
  --tarball ~/Downloads/nginx-release-${VERSION}.tgz \
  --version ${VERSION} --force
git add -N releases/
git add -p
git ci -v
git tag $VERSION
git push
git push --tags
```

Then draft a new release on GitHub: <https://github.com/cloudfoundry-community/nginx-release/releases>
