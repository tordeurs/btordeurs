---
title: Configuring Gradle for use with GitHub Packages
intro: 'You can configure Gradle to publish packages to {% data variables.product.prodname_registry %} and to use packages stored on {% data variables.product.prodname_registry %} as dependencies in a Java project.'
product: '{% data reusables.gated-features.packages %}'
redirect_from:
  - /articles/configuring-gradle-for-use-with-github-package-registry
  - /github/managing-packages-with-github-package-registry/configuring-gradle-for-use-with-github-package-registry
  - /github/managing-packages-with-github-packages/configuring-gradle-for-use-with-github-packages
  - /packages/using-github-packages-with-your-projects-ecosystem/configuring-gradle-for-use-with-github-packages
versions:
  free-pro-team: '*'
  enterprise-server: '>=2.22'
  github-ae: '*'
---

{% data reusables.package_registry.packages-ghes-release-stage %}
{% data reusables.package_registry.packages-ghae-release-stage %}

**Note:** When installing or publishing a docker image, {% data variables.product.prodname_registry %} does not currently support foreign layers, such as Windows images.

### Bei {% data variables.product.prodname_registry %} authentifizieren

{% data reusables.package_registry.authenticate-packages %}

#### Authenticating with a personal access token

{% data reusables.package_registry.required-scopes %}

You can authenticate to {% data variables.product.prodname_registry %} with Gradle using either Gradle Groovy or Kotlin DSL by editing your *build.gradle* file (Gradle Groovy) or *build.gradle.kts* file (Kotlin DSL) file to include your personal access token. You can also configure Gradle Groovy and Kotlin DSL to recognize a single package or multiple packages in a repository.

{% if enterpriseServerVersions contains currentVersion %}
Replace *REGISTRY-URL* with the URL for your instance's Maven registry. If your instance has subdomain isolation enabled, use `maven.HOSTNAME`. If your instance has subdomain isolation disabled, use `HOSTNAME/_registry/maven`. In either case, replace *HOSTNAME* with the host name of your
{% data variables.product.prodname_ghe_server %} instance.
{% elsif currentVersion == "github-ae@latest" %}
Replace *REGISTRY-URL* with the URL for your enterprise's Maven registry, `maven.HOSTNAME`. Replace *HOSTNAME* with the host name of
{% data variables.product.product_location %}.
{% endif %}

Replace *USERNAME* with your {% data variables.product.prodname_dotcom %} username, *TOKEN* with your personal access token, *REPOSITORY* with the name of the repository containing the package you want to publish, and *OWNER* with the name of the user or organization account on {% data variables.product.prodname_dotcom %} that owns the repository. Because uppercase letters aren't supported, you must use lowercase letters for the repository owner even if the {% data variables.product.prodname_dotcom %} user or organization name contains uppercase letters.

{% note %}

**Note:** {% data reusables.package_registry.apache-maven-snapshot-versions-supported %} For an example, see "[Configuring Apache Maven for use with {% data variables.product.prodname_registry %}](/packages/using-github-packages-with-your-projects-ecosystem/configuring-apache-maven-for-use-with-github-packages)."

{% endnote %}

##### Example using Gradle Groovy for a single package in a repository

```shell
plugins {
    id("maven-publish")
}

publishing {
    repositories {
        maven {
            name = "GitHubPackages"
            url = uri("https://{% if currentVersion == "free-pro-team@latest" %}maven.pkg.github.com{% else %}<em>REGISTRY-URL</em>{% endif %}/<em>OWNER</em>/<em>REPOSITORY</em>")
            credentials {
                username = project.findProperty("gpr.user") ?: System.getenv("<em>USERNAME</em>")
                password = project.findProperty("gpr.key") ?: System.getenv("<em>TOKEN</em>")
            }
        }
    }
    publications {
        gpr(MavenPublication) {
            from(components.java)
        }
    }
}
```

##### Example using Gradle Groovy for multiple packages in the same repository

```shell
plugins {
    id("maven-publish") apply false
}

subprojects {
    apply plugin: "maven-publish"
    publishing {
        repositories {
            maven {
                name = "GitHubPackages"
                url = uri("https://{% if currentVersion == "free-pro-team@latest" %}maven.pkg.github.com{% else %}<em>REGISTRY-URL</em>{% endif %}/<em>OWNER</em>/<em>REPOSITORY</em>")
                credentials {
                    username = project.findProperty("gpr.user") ?: System.getenv("<em>USERNAME</em>")
                    password = project.findProperty("gpr.key") ?: System.getenv("<em>TOKEN</em>")
                }
            }
        }
        publications {
            gpr(MavenPublication) {
                from(components.java)
            }
        }
    }
}
```

##### Example using Kotlin DSL for a single package in the same repository

```shell
plugins {
    `maven-publish`
}

publishing {
    repositories {
        maven {
            name = "GitHubPackages"
            url = uri("https://{% if currentVersion == "free-pro-team@latest" %}maven.pkg.github.com{% else %}<em>REGISTRY-URL</em>{% endif %}/<em>OWNER</em>/<em>REPOSITORY</em>")
            credentials {
                username = project.findProperty("gpr.user") as String? ?: System.getenv("<em>USERNAME</em>")
                password = project.findProperty("gpr.key") as String? ?: System.getenv("<em>TOKEN</em>")
            }
        }
    }
    publications {
        register<MavenPublication>("gpr") {
            from(components["java"])
        }
    }
}
```

##### Example using Kotlin DSL for multiple packages in the same repository

```shell
plugins {
  `maven-publish` apply false
  }

  subprojects {
  apply(plugin = "maven-publish")
  configure<PublishingExtension> {
  repositories {
  maven {
  name = "GitHubPackages"
  url = uri("https://maven.pkg.github.com/<em>OWNER</em>/<em>REPOSITORY</em>")
  credentials {
  username = project.findProperty("gpr.user") as String? ?: System.getenv("<em>USERNAME</em>")
  password = project.findProperty("gpr.key") as String? ?: System.getenv("<em>TOKEN</em>")
  }
  }
  }
  publications {
  register<MavenPublication>("gpr") {
  from(components["java"])
  }
  }
  }
  }
```

  #### Authenticating with the `GITHUB_TOKEN`

  {% data reusables.package_registry.package-registry-with-github-tokens %}

  For more information about using `GITHUB_TOKEN` with Maven, see "[Publishing Java packages with Maven](/actions/language-and-framework-guides/publishing-java-packages-with-maven#publishing-packages-to-github-packages)."

  ### Publishing a package

  {% data reusables.package_registry.default-name %} For example, {% data variables.product.prodname_dotcom %} will publish a package named `com.example.test` in the `OWNER/test` {% data variables.product.prodname_registry %} repository.

  {% data reusables.package_registry.viewing-packages %}

  {% data reusables.package_registry.authenticate-step %}
  2. After creating your package, you can publish the package.

   ```shell
   $ gradle publish
  ```

### Ein Paket installieren

You can install a package by adding the package as a dependency to your project. Weitere Informationen finden Sie unter ???[Declaring dependencies](https://docs.gradle.org/current/userguide/declaring_dependencies.html)??? (Abh??ngigkeiten festlegen) in der Gradle-Dokumentation.

{% data reusables.package_registry.authenticate-step %}
2. Add the package dependencies to your *build.gradle* file (Gradle Groovy) or *build.gradle.kts* file (Kotlin DSL) file.

  Example using Grady Groovy:
  ```shell
  dependencies {
  implementation 'com.example:package'
  }
  ```
  Example using Kotlin DSL:
  ```shell
  dependencies {
  implementation("com.example:package")
  }
  ```

3. Add the maven plugin to your *build.gradle* file (Gradle Groovy) or *build.gradle.kts* file (Kotlin DSL) file.

  Example using Grady Groovy:
  ```shell
  plugins {
  id 'maven'
  }
  ```
  Example using Kotlin DSL:
  ```shell
  plugins {
  `maven`
  }
  ```

  3. Installieren Sie das Paket.

  ```shell
  $ gradle install
  ```

### Weiterf??hrende Informationen

- "[Configuring Apache Maven for use with {% data variables.product.prodname_registry %}](/packages/using-github-packages-with-your-projects-ecosystem/configuring-apache-maven-for-use-with-github-packages)"
- "{% if currentVersion == "free-pro-team@latest" or currentVersion ver_gt "enterprise-server@3.0" %}[Deleting and restoring a package](/packages/learn-github-packages/deleting-and-restoring-a-package){% elsif currentVersion ver_lt "enterprise-server@3.1" or currentVersion == "github-ae@latest" %}[Deleting a package](/packages/learn-github-packages/deleting-a-package){% endif %}"
