## SAML (Security Assertion Markup Language) Setup and Configuration

### Configure Automatic Metadata Generation
* Open `securityContext.xml` file in an editor for the deployed component
  * **NOTE:** The `securityContext.xml` file can be found in `[Tomcat web application directory]/[component]/WEB-INF/classes/security`
    * Example: `/var/lib/tomcat7/webapps/ROOT/WEB-INF/classes/security/securityContext.xml`
  * **NOTE:** When editing the `securityContext.xml` file, elevated privileges (i.e. `sudo`) may by required
* Add the following line within a `<security:http>` element:
  * `<security:custom-filter before="FIRST" ref="metadataGeneratorFilter" />`
  * **NOTE:** Typically a `<security:http>` element can be found around line 31 of the `securityContext.xml` file
  * Example:

~~~~
<security:http entry-point-ref="delegatingAuthenticationEntryPoint" use-expressions="true">
    <security:custom-filter before="FIRST" ref="metadataGeneratorFilter" />
    <security:custom-filter ref="oauth2ProviderFilter" before="PRE_AUTH_FILTER" />
    <security:custom-filter after="BASIC_AUTH_FILTER" ref="samlFilter"/>
    <security:intercept-url pattern="/**" access="isFullyAuthenticated()"/>
</security:http>
~~~~

* Add configuration for the SAML metadata generator to `securityContext.xml`:
  * Add the following `<bean>` definitions to `securityContext.xml`:

~~~~
<bean id="metadataGeneratorFilter" class="org.springframework.security.saml.metadata.MetadataGeneratorFilter">
    <constructor-arg ref="metadataGenerator"/>
</bean>

<bean id="metadataGenerator" class="org.springframework.security.saml.metadata.MetadataGenerator">
    <property name="bindingsSSO">
        <list>
            <value>redirect</value>
            <value>artifact</value>
        </list>
    </property>
    <property name="entityId" value="[name of component]"/>
</bean>
~~~~

**NOTE:** The component name should not have spaces.  Shown below is an example of a `metadataGenerator` configured with an `entityId` of `progman_rest`:

~~~~
<bean id="metadataGeneratorFilter" class="org.springframework.security.saml.metadata.MetadataGeneratorFilter">
    <constructor-arg ref="metadataGenerator"/>
</bean>

<bean id="metadataGenerator" class="org.springframework.security.saml.metadata.MetadataGenerator">
    <property name="bindingsSSO">
        <list>
            <value>redirect</value>
            <value>artifact</value>
        </list>
    </property>
    <property name="entityId" value="progman_rest"/>
</bean>
~~~~

* Restart Tomcat:
  * `sudo service tomcat7 restart`

#### Verify SAML Metadata Setup
* Visit the `/saml/metadata` endpoint for the deployed component:
  * Example:  `http://54.213.81.243:8080/rest/saml/metadata`
* The output should appear as XML containing:
  * The X509 Certificate data
  * URLs containing the domain name of the server hosting the component as the value of a `Location` attribute
    * Examples:
      * `<md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="http://54.213.81.243:8080/rest/saml/SingleLogout"/>`
      * `<md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Artifact" Location="http://54.213.81.243:8080/rest/saml/SSO" index="0" isDefault="true"/>`

### SAML Pre-Configured Metadata Configuration
* Save the output of `/saml/metadata` endpoint to `/var/lib/tomcat/resources/security/`
  * Example: save the `54.213.81.243:8080/saml/metadata` output to `/var/lib/tomcat/resources/security/saml_metadata.xml`
* Disable (by removing or commenting out) the `<security:custom-filter before="FIRST" ref="metadataGeneratorFilter" />` from the `securityContext.xml` file to disable the autoamtic generation of SAML metadata
  * The automatic generation of SAML metadata is only needed once to generate the metadata file.  After the metadata file is generated, there is no further need for automatically generating SAML metadata.
* ***OPTIONAL:*** Remove the `metadataGeneratorFilter` and `metadataGenerator` bean definitions from the `securityContext.xml`
* Set permissions on the metadata XML file(s) so that only the `tomcat7` user can read it/them:
  * `sudo chmod 0400 /var/lib/tomcat/resources/security/*.xml`

#### Additional Notes
* [Spring SAML Metadata Configuration](http://docs.spring.io/spring-security-saml/docs/current/reference/html/configuration-metadata.html#configuration-metadata-sp-generation)