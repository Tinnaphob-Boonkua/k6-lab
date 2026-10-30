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
| A3 | Stress | 5 | | | | | |
| A4 | Spike | 5 | | | | | |
| A5 | Soak | 2 | | | | | |
## Part B
1.
2.
3.
4.
5.
6.
7.
8.
9.