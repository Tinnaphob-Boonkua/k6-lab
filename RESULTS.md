# K6 lab results
## Part 0
- Step 0.1, http_req_duration: avg=328.95ms min=328.95ms med=328.95ms max=328.95ms p(90)=328.95ms p(95)=328.95ms
- Step 0.2, requests, rate and p(95), and why the rate stays below 5:75 requests, 3.705553 req/s, p(95)=308.25 ms.
  The request rate stays below 5 req/s because each virtual user sleeps for 1 second after every request, and the HTTP request itself also takes about 0.3 seconds. Therefore, one iteration takes about 1.34 seconds on average, so 5 VUs cannot generate 5 requests every second.
- Step 0.6, exit code of the normal run and of the run with P95=300:  Normal run exit code: 0, P95=300 run exit code: 99
- Step 0.7, rules that passed, p(95), p(99), exit code after your change:
  Initial run: all rules passed.
  p(95)=411.49 ms, p(99)=417.36 ms, http_req_failed=0.00%.
  After changing p(95) threshold to 300 ms, the p(95) rule failed.
  Exit code: 99
- Step 0.8, what was wrong, and how the checks showed it:
  The excluded ingredient and tool names used the wrong capitalization.
  The script sent "pepperoni" and "knife", but the API uses "Pepperoni" and "Knife".
  Because the values are case-sensitive, the exclusions were not applied correctly.
  Under load, repeated requests produced enough different pizza recommendations for the checks to detect cases containing Pepperoni or requiring a Knife. A single manual request might randomly return a valid pizza and hide the problem.
## Part A results
| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1 | Smoke | 1 | 0.519792/s | 413.15ms | 417.4ms | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.29311/s | 413.05ms | 429.72ms | 0.00% | 100.00% |
| A3 | Stress | 5 | 2.089879/s | 431.33ms | 450.98ms | 0.00% | 100.00% |
| A4 | Spike | 5 | 0.980081/s | 410.3ms | 453.07ms | 0.00% | 100.00% |
| A5 | Soak | 2 | 0.959505/s | 421.53ms | 1.3s | 0.00% | 100.00% |
## Part B
1.The results for the smoke, average-load, stress, spike, and soak tests are shown in the Part A results table above. All five tests passed their thresholds, with 0.00% failed requests and 100.00% successful checks.
2.In Step 0.2, the request rate was below 5 requests per second even though 5 virtual users were running because each virtual user waits for the HTTP request to finish and then sleeps for 1 second. In my run, the average iteration duration was about 1.34 seconds. Therefore, each VU could not send one request every second. The measured request rate was 3.705553 requests/s.
3.Yes, the response time increased slightly from the average-load test to the stress test. The p(95) increased from 413.05 ms in A2 to 431.33 ms in A3, an increase of 18.28 ms. However, the failure rate remained at 0.00% and all thresholds still passed. This suggests that increasing the load from 3 to 5 users caused a small slowdown, but the system was still operating within the tested operational envelope.
4.Yes, the response time increased during the spike. Before the spike, the p(95) response time was around 370–390 ms. When the load increased to 5 users, p(95) rose to approximately 430–445 ms. After the load dropped back toward 1 user, the response time returned to approximately its earlier level within about 15–20 seconds. No requests failed during the spike.
5.The response time and error rate did not get progressively worse during the soak test. For most of the test, the response time remained at approximately the same level from the beginning to the end. There were two short response-time spikes during the run, but the values quickly returned to their normal range afterward, so they do not indicate continuing degradation. The overall p(95) was 421.53 ms, the failed-request rate remained at 0.00%, and all checks passed. A longer soak test lasting several hours could reveal defects such as a memory leak, where resource usage gradually increases and eventually causes the system to slow down or fail.
6.The thresholds use p(95) instead of the average because an average can hide a small number of very slow requests. The p(95) value shows the response time within which 95% of requests completed, so it gives a better indication of the experience of slower users.
7.The same tests could produce different results in a laboratory environment for at least two reasons. First, QuickPizza is accessed through the public internet, so network latency, routing, and congestion can affect response times. Second, QuickPizza is a shared public service, so other users may generate additional server load while the test is running. A controlled laboratory environment could provide a more stable network and a controlled server workload.
8.A stress, spike, or breakpoint test with hundreds of users could overload the target system and disrupt the service for real users. It should never be performed against a site that I do not own or do not have permission to test because it creates a large amount of unwanted traffic and could behave similarly to a denial-of-service attack.
9.Vague performance requirement:

"The application should respond quickly."

Testable performance requirement:

"With 20 concurrent users for 2 minutes, 95% of requests must complete within 1 second, 99% of requests must complete within 1.5 seconds, and fewer than 1% of requests may fail."

K6 thresholds:

thresholds: {
  http_req_duration: [
    'p(95)<1000',
    'p(99)<1500',
  ],
  http_req_failed: ['rate<0.01'],
}