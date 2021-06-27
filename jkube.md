## Configure OKD 3.11 For Jkube Deployment Access

After install OKD 3.11 you could encounter some problems deploying with the Jkube Plugin, so take on this steps in the Okd cluster for access the internal registry and push images from the plugin.

~~~bash
oc login -u system:admin
oc policy add-role-to-user registry-viewer developer
oc policy add-role-to-user registry-editor developer
oc project default
oc adm policy add-cluster-role-to-user system:controller:replication-controller -z deployer
oc rollout latest dc/docker-registry
oc rollout latest dc/router
Check the changes Applied 
get pods --all-namespaces -o wide
~~~

If you use the Maven Plugin to deploy Java Aplications you could use this configuration in case of error

~~~xml
<plugin>
   <groupId>org.eclipse.jkube</groupId>
   <artifactId>openshift-maven-plugin</artifactId>
   <version>1.0.0-alpha-3</version>
   <configuration>
      <generator>
         <includes>
            <include>quarkus</include>
         </includes>
         <config>
            <quarkus>
               <from>registry.access.redhat.com/openjdk/openjdk-11-rhel7</from>
            </quarkus>
         </config>
      </generator>
   </configuration>
</plugin>
~~~

This configuration override the image since the default image openjdk:11 doesn't contains the configuration for user.

Here is a complete example:

~~~xml
<plugin>
   <groupId>org.eclipse.jkube</groupId>
   <artifactId>openshift-maven-plugin</artifactId>
   <version>1.0.0-alpha-3</version>
   <configuration>
      <images>
         <image>
            <name>${project.groupId}/${project.artifactId}</name>
            <build>
               <from>registry.access.redhat.com/openjdk/openjdk-11-rhel7</from>
               <tags>
                  <tag>latest</tag>
                  <tag>${project.version}</tag>
               </tags>
               <env>
                  <JAVA_APP_JAR>${project.artifactId}-${project.version}-runner.jar</JAVA_APP_JAR>
                  <JAVA_OPTIONS>-Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager</JAVA_OPTIONS>
               </env>
               <assembly>
                  <mode>dir</mode>
                  <targetDir>/</targetDir>
                  <inline>
                     <id>customized-quarkus</id>
                     <files>
                        <file>
                           <source>target/${project.artifactId}-${project.version}-runner.jar</source>
                           <outputDirectory>.</outputDirectory>
                        </file>
                     </files>
                     <fileSets>
                        <fileSet>
                           <directory>${project.basedir}/target/lib</directory>
                           <outputDirectory>.</outputDirectory>
                           <fileMode>755</fileMode>
                        </fileSet>
                     </fileSets>
                  </inline>
               </assembly>
               <ports>8080</ports>
               <user>185</user>
            </build>
         </image>
      </images>
      <enricher>
         <config>
            <jkube-service>
               <type>NodePort</type>
            </jkube-service>
         </config>
      </enricher>
   </configuration>
</plugin>
~~~

