# Traderev Analytics

The Traderev analytics application consists of 4 components:

1. The static front-end (prod hostname: analytics.traderev.com, repo: https://github.com/TradeRev/tr-analytics)

2. The API backend (prod hostname: analytics-api.traderev.com, repo: https://github.com/TradeRev/tr-analytics-server-grails)

3. A reporting application that queries databases and sends reports out via email (repo: https://github.com/TradeRev/analytics-server)

4. A set of lambda functions that will POST to the reporting application (component #3) and a set of cloudwatch events that will call the lambda functions according to a schedule (repo: https://github.com/TradeRev/devops (tf-analytics terraform module))

Infrastructure Details
----------------------
1. The static front-end is hosted in an S3 bucket.  To deploy the content, clone the repository on a system with Node.js installed.  Then run cd into the repo directory and run 'npm install', then 'bower install'.  Edit gulpfile.js and ensure that the correct bucket names and endpoints are associated with each environment.  
