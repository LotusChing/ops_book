# 求助

Github上看到的，记录下以后如果遇到问题求助别人时参考对照

Please follow these instructions to help us diagnose your issue

1. create a new issue, with a succinct title that describes your issue:
  * bad title: "It doesn't work with my docker"
  * good title: "Private registry push fail: 400 error with E_INVALID_DIGEST"
2. copy the output of:
  * docker version
  * docker info
  * docker exec <registry-container> registry -version
3. copy the command line you used to launch your Registry
4. restart your docker daemon in debug mode (add -D to the daemon launch arguments)
5. reproduce your problem and get your docker daemon logs showing the error
6. if relevant, copy your registry logs that show the error
7. provide any relevant detail about your specific Registry configuration (e.g., storage backend used)
8. indicate if you are using an enterprise proxy, Nginx, or anything else between you and your Registry