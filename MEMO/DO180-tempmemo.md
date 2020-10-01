[student@workstation DO180-apps]$ oc logs --all-containers -f php-helloworld-part2-1-build
Caching blobs under "/var/cache/blobs".
Getting image source signatures
Copying blob sha256:e7021e0589e97471d99c4265b7c8e64da328e48f116b5f260353b2e0a2adb373
Copying blob sha256:fc5b206e9329a1674dd9e8efbee45c9be28d0d0dcbabba3c6bb67a2f22cfcf2a
Copying blob sha256:9e7a6dc796f0a75c560158a9f9e30fb8b5a90cb53edce9ffbdf5778406e4de39
Copying blob sha256:f659c5c779ac4373302bfe3dc7d713c59cf9ec9f179a71e9b26336a51043fad2
Copying blob sha256:14e4b218199e7985dbeca760da0fa9b969ca5fdaae5095a273cd924afa6898e9
Copying config sha256:988e5d4c035d1585f006d0c57a0cfe4bc9f8aae412585f4a6b8fed87da45105f
Writing manifest to image destination
Storing signatures
Generating dockerfile with builder image image-registry.openshift-image-registry.svc:5000/openshift/php@sha256:962d936b00f4953d977be9b85268a289bf2f9d49df4d51b408a7c7835a551103
STEP 1: FROM image-registry.openshift-image-registry.svc:5000/openshift/php@sha256:962d936b00f4953d977be9b85268a289bf2f9d49df4d51b408a7c7835a551103
STEP 2: LABEL "io.openshift.build.commit.date"="Fri Oct 4 13:04:03 2019 +0200"       "io.openshift.build.commit.id"="f7cd8963ef353d9173c3a21dcccf402f3616840b"       "io.openshift.build.commit.ref"="s2i"       "io.openshift.build.commit.message"="Initial commit, including all apps previously in course"       "io.openshift.build.source-location"="https://github.com/Yasunari0118/DO180-apps"       "io.openshift.build.source-context-dir"="php-helloworld"       "io.openshift.build.image"="image-registry.openshift-image-registry.svc:5000/openshift/php@sha256:962d936b00f4953d977be9b85268a289bf2f9d49df4d51b408a7c7835a551103"       "io.openshift.build.commit.author"="Jordi Sola <someth2say@gmail.com>"
STEP 3: ENV OPENSHIFT_BUILD_NAME="php-helloworld-part2-1"     OPENSHIFT_BUILD_NAMESPACE="yasunari-tokutake-s2i"     OPENSHIFT_BUILD_SOURCE="https://github.com/Yasunari0118/DO180-apps"     OPENSHIFT_BUILD_REFERENCE="s2i"     OPENSHIFT_BUILD_COMMIT="f7cd8963ef353d9173c3a21dcccf402f3616840b"
STEP 4: USER root
STEP 5: COPY upload/src /tmp/src
STEP 6: RUN chown -R 1001:0 /tmp/src
STEP 7: USER 1001
STEP 8: RUN /usr/libexec/s2i/assemble
---> Installing application source...
=> sourcing 20-copy-config.sh ...
---> 08:25:36     Processing additional arbitrary httpd configuration provided by s2i ...
=> sourcing 00-documentroot.conf ...
=> sourcing 50-mpm-tuning.conf ...
=> sourcing 40-ssl-certs.sh ...
STEP 9: CMD /usr/libexec/s2i/run
STEP 10: COMMIT temp.builder.openshift.io/yasunari-tokutake-s2i/php-helloworld-part2-1:f436f207
time="2020-10-01T08:25:36Z" level=info msg="Image operating system mismatch: image uses OS \"\"+architecture \"\", expecting one of \"linux+amd64\""
Getting image source signatures
Copying blob sha256:abc8c32bad762f509f729872b552e8366b9c71a863dc5e9e648dda04a7896e50
Copying blob sha256:b9e30d7689db4040323cb65632888c37d96cbd60d0f05c4dd37e78b9b8a0566a
Copying blob sha256:8b0dba9179a6a0d5de65c6b001c8c9c83eaa169ce526b22a33845ede8f74568c
Copying blob sha256:bda7ec697f579121100fed608753105e2150894241b011af8b12fea98296f8f7
Copying blob sha256:5daa741fd16f3d131c8667e9e7bcbcb0ce95243275070d34686eb6df2c6200fd
Copying blob sha256:d23603393487f7b9d69997569b75300b8a97a7cd9d711f85777ad9307a2a3722
Copying config sha256:1f339589e2e1c6545157d64f4ec5aaccb2341a2c17d6d624596bdd278f57ea26
Writing manifest to image destination
Storing signatures
--> 1f339589e2e
1f339589e2e1c6545157d64f4ec5aaccb2341a2c17d6d624596bdd278f57ea26

Pushing image image-registry.openshift-image-registry.svc:5000/yasunari-tokutake-s2i/php-helloworld-part2:latest ...
Getting image source signatures
Copying blob sha256:d23603393487f7b9d69997569b75300b8a97a7cd9d711f85777ad9307a2a3722
Copying blob sha256:14e4b218199e7985dbeca760da0fa9b969ca5fdaae5095a273cd924afa6898e9
Copying blob sha256:fc5b206e9329a1674dd9e8efbee45c9be28d0d0dcbabba3c6bb67a2f22cfcf2a
Copying blob sha256:f659c5c779ac4373302bfe3dc7d713c59cf9ec9f179a71e9b26336a51043fad2
Copying blob sha256:9e7a6dc796f0a75c560158a9f9e30fb8b5a90cb53edce9ffbdf5778406e4de39
Copying blob sha256:e7021e0589e97471d99c4265b7c8e64da328e48f116b5f260353b2e0a2adb373
Copying config sha256:1f339589e2e1c6545157d64f4ec5aaccb2341a2c17d6d624596bdd278f57ea26
Writing manifest to image destination
Storing signatures
Successfully pushed image-registry.openshift-image-registry.svc:5000/yasunari-tokutake-s2i/php-helloworld-part2@sha256:4fec8db0c93e61cb563deadbdf61cbfe163d3168b2cdfbd333c4582c80948e1c
Push successful
Cloning "https://github.com/Yasunari0118/DO180-apps" ...
	Commit:	f7cd8963ef353d9173c3a21dcccf402f3616840b (Initial commit, including all apps previously in course)
	Author:	Jordi Sola <someth2say@gmail.com>
	Date:	Fri Oct 4 13:04:03 2019 +0200
[student@workstation DO180-apps]$ oc describe dc/php-helloworld-part2-1-build
Error from server (NotFound): deploymentconfigs.apps.openshift.io "php-helloworld-part2-1-build" not found
[student@workstation DO180-apps]$ oc describe dc/php-helloworld-part2
Name:		php-helloworld-part2
Namespace:	yasunari-tokutake-s2i
Created:	2 minutes ago
Labels:		app=php-helloworld-part2
		app.kubernetes.io/component=php-helloworld-part2
		app.kubernetes.io/instance=php-helloworld-part2
Annotations:	openshift.io/generated-by=OpenShiftNewApp
Latest Version:	1
Selector:	deploymentconfig=php-helloworld-part2
Replicas:	1
Triggers:	Config, Image(php-helloworld-part2@latest, auto=true)
Strategy:	Rolling
Template:
Pod Template:
  Labels:	deploymentconfig=php-helloworld-part2
  Annotations:	openshift.io/generated-by: OpenShiftNewApp
  Containers:
   php-helloworld-part2:
    Image:		image-registry.openshift-image-registry.svc:5000/yasunari-tokutake-s2i/php-helloworld-part2@sha256:4fec8db0c93e61cb563deadbdf61cbfe163d3168b2cdfbd333c4582c80948e1c
    Ports:		8080/TCP, 8443/TCP
    Host Ports:		0/TCP, 0/TCP
    Environment:	<none>
    Mounts:		<none>
  Volumes:		<none>

Deployment #1 (latest):
	Name:		php-helloworld-part2-1
	Created:	about a minute ago
	Status:		Complete
	Replicas:	1 current / 1 desired
	Selector:	deployment=php-helloworld-part2-1,deploymentconfig=php-helloworld-part2
	Labels:		app.kubernetes.io/component=php-helloworld-part2,app.kubernetes.io/instance=php-helloworld-part2,app=php-helloworld-part2,openshift.io/deployment-config.name=php-helloworld-part2
	Pods Status:	1 Running / 0 Waiting / 0 Succeeded / 0 Failed

Events:
  Type		Reason			Age	From				Message
  ----		------			----	----				-------
  Normal	DeploymentCreated	105s	deploymentconfig-controller	Created new replication controller "php-helloworld-part2-1" for version 1
[student@workstation DO180-apps]$ oc expose svc php-helloworld-part2-1-build
Error from server (NotFound): services "php-helloworld-part2-1-build" not found
[student@workstation DO180-apps]$ oc expose svc php-helloworld-part2
route.route.openshift.io/php-helloworld-part2 exposed
[student@workstation DO180-apps]$ oc get expose
error: the server doesn't have a resource type "expose"
[student@workstation DO180-apps]$ oc get route
NAME                   HOST/PORT                                                               PATH   SERVICES               PORT       TERMINATION   WILDCARD
php-helloworld-part2   php-helloworld-part2-yasunari-tokutake-s2i.apps.ap45.prod.nextcle.com          php-helloworld-part2   8080-tcp                 None
[student@workstation DO180-apps]$ curl php-helloworld-part2-yasunari-tokutake-s2i.apps.ap45.prod.nextcle.com
Hello, World! php version is 7.3.11
[student@workstation DO180-apps]$ 
[student@workstation DO180-apps]$ curl php-helloworld-part2-yasunari-tokutake-s2i.apps.ap45.prod.nextcle.com
Hello, World! php version is 7.3.11
[student@workstation DO180-apps]$ cd php-helloworld/
[student@workstation php-helloworld]$ vi index.php 
[student@workstation php-helloworld]$ git add .
[student@workstation php-helloworld]$ git commit 
Aborting commit due to empty commit message.
[student@workstation php-helloworld]$ git commit -m "Changed"
[s2i 776c5a5] Changed
 1 file changed, 1 insertion(+)
[student@workstation php-helloworld]$ git push origin s2i
Counting objects: 7, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 403 bytes | 0 bytes/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/Yasunari0118/DO180-apps
   f7cd896..776c5a5  s2i -> s2i
[student@workstation php-helloworld]$ oc start-build php-helloworld-part2
build.build.openshift.io/php-helloworld-part2-2 started
[student@workstation php-helloworld]$ oc get build
NAME                     TYPE     FROM          STATUS                             STARTED          DURATION
php-helloworld-1         Source   Git@f7cd896   Failed (InvalidContextDirectory)   13 minutes ago   7s
php-helloworld-part2-1   Source   Git@f7cd896   Complete                           7 minutes ago    38s
php-helloworld-part2-2   Source   Git@776c5a5   Running                            29 seconds ago   
[student@workstation php-helloworld]$ oc get pods -w
NAME                            READY   STATUS              RESTARTS   AGE
php-helloworld-part2-1-build    0/1     Completed           0          7m24s
php-helloworld-part2-1-deploy   0/1     Completed           0          6m45s
php-helloworld-part2-1-qlnvm    1/1     Running             0          6m42s
php-helloworld-part2-2-build    0/1     Completed           0          40s
php-helloworld-part2-2-deploy   0/1     ContainerCreating   0          1s
php-helloworld-part2-2-deploy   0/1     ContainerCreating   0          2s
php-helloworld-part2-2-deploy   1/1     Running             0          2s
php-helloworld-part2-2-zgzm5    0/1     Pending             0          0s
php-helloworld-part2-2-zgzm5    0/1     Pending             0          0s
php-helloworld-part2-2-zgzm5    0/1     ContainerCreating   0          0s
php-helloworld-part2-2-zgzm5    0/1     ContainerCreating   0          3s
php-helloworld-part2-2-zgzm5    1/1     Running             0          5s
php-helloworld-part2-1-qlnvm    1/1     Terminating         0          6m49s
php-helloworld-part2-1-qlnvm    0/1     Terminating         0          6m50s
php-helloworld-part2-1-qlnvm    0/1     Terminating         0          6m51s
php-helloworld-part2-1-qlnvm    0/1     Terminating         0          6m51s
php-helloworld-part2-2-deploy   0/1     Completed           0          12s
^C[student@workstation php-helloworld]curl php-helloworld-part2-yasunari-tokutake-s2i.apps.ap45.prod.nextcle.com
Hello, World! php version is 7.3.11
A change is coming
