# Upgrading the Capstone codebase from Ruby 2.7 to Ruby 3.0
Capstone is a monolithic Ruby on Rails application. In order for Capstone to continue receiving critical security updates, it must run on Ruby 3.0. Ruby 2.7 reaches end-of-life on 2023-03-31. Failure to upgrade Capstone to Ruby 3.0 before the end-of-life date may expose the application to security risks, as Ruby maintainers will cease to release fixes for version 2.7.

## Audience
This guide is meant for the Capstone platform engineering team.

## Prerequisites
Platform engineers should be familiar with the following:
- the language changes introduced by Ruby 3.0 (see [Ruby 3.0 Language Changes](https://rubyreferences.github.io/rubychanges/3.0.html) for more information)
- how gems are managed and updated
- running Capstone locally
- running unit tests and end-to-end tests
- [rolling back deployments](#)

## Ensuring Gem compatibility
Capstone uses many third-party gems, all of which are specified in the application’s `Gemfile`. Ensure that each gem is compatible with Ruby 3.0 by checking the documentation of the gem and verifying that no deprecation warnings are raised from the gem code. It is possible that a gem is no longer maintained and therefore not updated to be compatible with Ruby 3.0. For such a case, identify where in the Capstone codebase the gem is used and contact the relevant code owners to decide the best course of action, which may include [monkeypatching](https://blog.appsignal.com/2021/08/24/responsible-monkeypatching-in-ruby.html) or finding a replacement gem.

Ruby 3.0 also introduces changes in the standard library. This change means that some gems may need to have their references added to the `Gemfile`, while other gems may need to have their references removed. See [stdlib compatibility issues](https://github.com/ruby/ruby/blob/v3_0_0/NEWS.md#stdlib-compatibility-issues) for a list of affected gems.

## Fixing deprecation warnings
To automatically generate deprecation warnings and fix all of them, consider following the approach detailed in [How we automatically fixed thousands of Ruby 2.7 deprecation warnings](https://about.gitlab.com/blog/2021/02/03/how-we-automatically-fixed-hundreds-of-ruby-2-7-deprecation-warnings/).

Take the following steps when creating pull requests for the code changes:
1. Organize modified files based on their code owners, identified by the `@team` file annotation at the top of each file.
2. Create a pull request for each code owner and assign the pull request to the respective code owner.
3. Make any changes and address questions as necessary, following the guidance provided by the code owner.
4. Once the pull request is approved, merge it. Note that these fixes are expected to be compatible with both Ruby 3.0 and Ruby 2.7, allowing them to be safely pushed even though the application is still on Ruby 2.7 at this point.

## Running Capstone on Ruby 3.0 locally
1. Update the `.ruby-version` file and Gemfile to specify Ruby 3.0.
2. Run Capstone locally and manually test through some basic workflows of the application. This manual testing is important because the generation of deprecation warnings relies on test coverage, which may not capture all workflows.
3. Run all tests locally.
4. If errors are encountered in steps 2 and 3, fix the errors and repeat the steps until no more errors are encountered.
5. Submit a pull request containing the version upgrade change along with any necessary changes that arise due to testing. Assign the pull request to the platform engineering team and the relevant code owners based on the files that were modified.
6. Make any changes and address questions on the pull request as necessary.

Once the pull request is approved, the application is ready to go live with the upgraded Ruby version in the staging environment.

## Deploy to staging
Once the pull request that explicitly upgrades the application code version to Ruby 3 is merged, it will automatically be deployed to staging. Monitor the deployment for any potential issues, and in case of critical functionality failures or other major issues, be prepared to roll back the deployment. Once the updated code is in staging, ask all code owners to manually test workflows that they are familiar with to verify that all functionality works as expected. Ensure that all tests pass on staging.

## Deploy to production
At this point, the Ruby 3.0 upgrade appears stable on staging and can be promoted to production with a high degree of confidence. The code in staging will be promoted to production according to the [deployment schedule](#). During this time, monitor the deployment and repeat the same testing that was done on staging for production. Be prepared to roll back the deployment if necessary.

## Conclusion
In conclusion, the upgrade of the Capstone codebase to Ruby 3.0 is a critical step for security and compatibility. Thorough testing in staging provides confidence in the stability of the update, with a contingency plan for potential issues. This upgrade is a result of collaborative efforts and ensures Capstone is aligned with the improvements introduced by Ruby 3.0, enhancing ongoing support.
