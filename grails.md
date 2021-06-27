## Deploy Grails 4 Applications in heroku

The official documentation is a little outdated for deploying Grails applications on Heroku, so this is recent approach to deploy your Grails Applications on Heroku.

Grails 4 can generate already a jar application with Spring Boot, so we no need to use the war approach.

In the build.gradle file add the stage task, this task is used by Heroku to deploy Gradle Applications:

~~~groovy
task stage() {
 dependsOn bootJar
}

And clean unused resources

//clean unneeded build artifacts
tasks.stage.doLast() {
 delete fileTree(dir: "build/assets")
 delete fileTree(dir: "build/classes")
 delete fileTree(dir: "build/gsp-classes")
 delete fileTree(dir: "build/gsptmp")
 delete fileTree(dir: "build/resources")
 delete fileTree(dir: "build/tmp")
}
~~~


The task bootJar is already configured so this can generate our jar standalone application.

The next thing to do is create a Procfile which contains the command for run the application. This file must contain the following content:

    web: java -jar build/libs/*.jar $JAVA_OPTS --server.port=$PORT

With the free tier of heroku is limited the amount of ram, for not overloading the requested ram, is recomended add a file java_options.env with all the java options variables, for example:

    JAVA_OPTS=-Xms256m -Xmx512m

Once Configured the appplication then do the normal steps for deploying apps in heroku

Test if the configuration was applied successfully:

~~~bash
heroku local web
heroku create
git push heroku master
heroku logs --tail
~~~

You can find all the configurations applied in my personal project: https://github.com/jorgecastro05/valueme
