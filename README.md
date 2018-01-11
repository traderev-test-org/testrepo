# Traderev Analytics

The Traderev analytics application consists of 4 components:

1. The static front-end (hosted on S3.  Prod hostname is analytics.traderev.com)

2. The API backend (prod hostname is analytics-api.traderev.com)

3. A reporting application that queries databases and sends reports out via email

4. A set of lambda functions that will POST to the reporting application (component #3) and a set of cloudwatch events that will call the lambda functions according to a schedule



