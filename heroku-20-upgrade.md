# Upgrading the Capstone servers from Heroku-18 to Heroku-20
Capstone is a monolithic web application that is hosted on Heroku. Heroku uses [stacks](https://devcenter.heroku.com/articles/stack), which are operating system images that provide the underlying setup necessary to run applications. Heroku-18 is a stack that reaches end-of-life on 2023-04-30. After this date, it will no longer be possible to deploy using the Heroku-18 stack. In order to ensure that Capstone deployments are still possible, the application must be upgraded to use the Heroku-20 stack before the end-of-life deadline.

## Audience
This guide is meant for the Capstone platform engineering team.

## Prerequisites
Platform engineers should be familiar with the following:
- a basic understanding of Heroku and its role in the Capstone platform
- using the Heroku command line interface
- making infrastructure changes via Terraform

## Ensuring Heroku-20 compatibility
Review the [Heroku-20 upgrade notes](https://devcenter.heroku.com/articles/heroku-20-stack#upgrade-notes) to understand the changes that are introduced in the newer stack. Follow the guidance in the document to ensure that Capstone will remain functional when using the Heroku-20 stack.

## Updating buildpacks
[Buildpacks](https://www.heroku.com/elements/buildpacks) are scripts that are used for app compilation. A common issue encountered when upgrading the Heroku stack is the use of [outdated buildpack versions](https://devcenter.heroku.com/articles/upgrading-to-the-latest-stack#old-buildpack-versions). Take the following steps to ensure buildpack compatibility:
1. Identify the buildpacks used in Capstone by navigating to the `capstone/staging/main.tf` file and finding the `buildpacks` argument in `resource “heroku_build”`. 
2. For each buildpack, review its documentation to check if its current version is compatible with Heroku-20.
3. If updating one or more buildpacks is necessary, then do so by changing the version number specified in the each buildpack URL (i.e. `https://github.com/heroku/buildpacks-ruby#v2.0.0`) to a compatible version and submit a pull request with the changes. Assign the pull request to the Capstone platform engineering team and merge the pull request when approved.
4. Repeat steps 1-3 for the buildpacks in `capstone/production/main.tf`. This step is to ensure that the buildpacks are up-to-date on both the staging and the production environments.

## Upgrading the stack version
To upgrade the Heroku stack version, do the following:
1. Locate the `stack` attribute under the `”heroku_app”` resource in the `capstone/staging/main.tf` file.
2. Update the value of the attribute to `”heroku-20”`.
3. Submit a pull request with the change and assign the pull request to the platform engineering team.
4. Once the pull request is approved, merge it.
5. Log in to the Terraform website and navigate to the `capstone-staging` workspace. 
6. Verify that the latest plan includes the changes to the stack version as well as any necessary buildpack version updates. 
7. Apply the plan and verify that it runs successfully.
8. If the plan fails to apply, review the logs outputted on the Terraform website and debug as necessary.
9. Once the plan on Terraform is applied, make the changes go into effect by restarting the Capstone application with the following command line interface instruction: `heroku restart -a capstone-staging`.
10. Verify that Capstone is stable on staging and that it is [using the Heroku-20 stack](https://devcenter.heroku.com/articles/stack#viewing-which-stack-your-app-is-using).
11. Repeat steps 1-8 using the `capstone/production/main.tf` file, the `capstone-production` workspace, and the `heroku restart -a capstone-production` command. This is to apply the changes to the production environment.

## Conclusion
In summary, the outlined steps provide a clear and systematic approach for the Capstone platform engineering team to successfully upgrade from Heroku-18 to Heroku-20. By addressing buildpack compatibility, submitting pull requests, and meticulously applying changes in both staging and production environments, this upgrade ensures the continued functionality and stability of the Capstone application. The proactive nature of this process mitigates risks associated with the end-of-life of Heroku-18, safeguarding the reliability of the web application.
