Q. what is docker?
- docker is platform  like github , in github we store codebase and in docker we store containers,also can say a containerization platform. along with storing, docker also provide some out of the box feature like networking , multi-stage build and so on.  


Q. how conatiner are diff from VM?
- the 1st basic diff is container came with a small os were as in VM we install the complete os,which make cotainer very light weight in nature also , bcz of complete os in vm it inc the risk of security as there were multiple software , so it will be easy for the hacker to find any vulnerability in VM then Containers.also conatiner have shared lib where as vm have complete lib from kernal to everything.


Q. what is docker lifecycle?
- user can create a dockerfile with the set of rule/command and can define their docker image, now image have all the set of instruction to build a container , so can select which base image they want and can also have all the dependencies of their need,further on once our conatiner is being working then we can push it to docker registry  and later all this as per their convince they can delete or down any conatiner from cli only.

Q. what are the different Docker Components?
- their are so many docker components, some of them were docker deamon -> is to execute all the actions, docker registry -> to store all the conatiner which you build , docker networking -> use to connect your conatiner through internet using virtual eth, docker cli -> to install or to perform anything from cli

Q. what is the diff between docker COPY  and docker ADD?
- COPY cmd is used in Dockerfile to copy files from any specific location (specifically host system) which were req in the respective container, where as ADD command is use to copy the files from a url.

Q. #what is diff between CMD and Entrypoint in Docker?
- we use CMD to run our application , suppose we have a python app so to run it we need to write run command which start with "python", so CMD mention that. were as Entrypoint are a place form where we run have to run our container.

Q. what are the networking types in Docker and what is the default?
- there are basically 3 types of networking we used in docker
1. bridge network 2. host network  3. ovelay netwrok
there is also a 4th one whic is MacVlan , which we don't use genrally.
were as Bridge network is the default network type.

Q. can you explain how to isolate networking between the containers?
- yeah we can use bridge network to diff diff container to get connected to the host network,were we can create our own bridge network which provide proper isolation here and no container can talk to the other container without any prior permission.

Q. What is multi stage build in Docker?
- so, when we build any docker image for a specific project which has some my extra thing which were just req for the build but we don't req them for the final output and bcz of that the size of file can go upto to 800mb - 1.5gb,and the concept of docker came into the bcz they are lightweighted but here the size is too big, which is too much so to fix this issue to reduce the image size, we use multi stage build, here what we do intially we build our image as we do inatally but rather then finishing the Dockerfile into 2 or more parts, where the we fetch the binary data for 1st build part which were only req to run this project and build that project with an destroless image, which eventually reduce the size of image to 1mb to 150 mb.

Q. what are destroless images in Docker?
- Destroless images are very lightweight weight images, which have nothing in them, just have small runtime dependencies or specific packages with a very minimun os lib which we req for our application, due to its lightweight nature we use them in multi-stage build.Also they are very very very secure as comapre to container,as they have very less file so getting exposed to  
vulnerability so chances of getting attacked by hacker get very low.


Q. Real Time challenges with Docker?

- Docker is a single daemon process. Which can cause a single point of fialure. if the docker daemon goes down for some reason all the app are down too, some morden solution to solve these issue , we can use PODMAN

- Docker Daemon run as a root user, which is a security threat. Any process running as a root can have adverse effect. When it is comprised for securty reasons, it can impact other application or containers on the host.

- Resiurce Constraints: if you're running too many conatiners on a single host, you may exp issues with resource constraints. This can result in slow performance or crashes. 

Q. What steps would you take to secure containers?

- Use Distroless or images with not to many packages as your final image in multi stage build, so that there is less chance of CVE or security issues.
- Ensure that the networking is config properly, this is one of the most common reasons for security issues. if required config custrom bridge network as assign them to isolate conatiners.
- Use utilities like Sync to scan your container images.