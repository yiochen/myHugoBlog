+++
title = "On Interview"
bigimg = ""
subtitle = "Things I learned from conducting technical interviews"
draft = false
categories = []
tags = []
date = "2017-11-20T23:15:09-08:00"

+++

I have done a couple technical interviews, both on-site and online. In my spare time, I also did
some mock interviews for friends. I am a recent grad. Not long ago, I was also going through the
grill of interviews. Now that I have sat at the other side of the table, I feel I learned more about
interviews.

> Disclaimer: the opinions in this article are my personal opinions. They don't reflect my company's
> and my team's hiring guideline.

## Choosing the coding question

One of the steps of preparing an interview is choosing a coding question for the interviewee. This
is a difficult task. Based on the limited information, I need to gauge the interviewee's capability
and choose a question just hard enough for interviewee to finish in 45 minutes. The question
wouldn't be too simple, because if we don't trust the interviewee's ability, we wouldn't invite
him/her for the technical interview. It wouldn't be too hard either because we only have about 45
minutes. Most people say the difficulty is similar to easy/median difficulty on
[LeetCode](leetcode.com).

> If the interviewee has a stronger background, s/he will get a harder question

It might sounds unfair, but it does make sense. If the interviewee finish the question with half the
time, the only information we get is: s/he passes the base hiring requirement for technical skill.
We failed at fully understand the candidate's capability. It's important to get a full picture of
the interviewee's problem solving skill, because when we are making the final decision on the
candidate, we will consider the feedback from all the interviews including some behaviors
interviews. Strong technical skills might make up for, for example, lack of working experience.

> Make the question as simple as possible

I feel the best questions are very easy to be understood, but challenging to solve. When the
interviewee spent 20 minutes to come up with the algorithm, and found that s/he misunderstood the
question, it suddenly puts a lot of pressure on the interviewee. For interviewer, it's also a waste
of time.

> Which algorithm or data structure to test on

Personally I prefer questions that requires only basic algorithm, I will not ask interviewee on
dynamic programming, binary search tree, etc. I am a front end developer. I feel in my daily work, I
seldom use those algorithm. I spent more time thinking about massaging data, storing data and
writing clean code. I want to test interviewees on those area. I will ask the interviewee some
questions that might be cumbersome to solve, so that they need to be careful about coding styles. If
they are tripping themselves over with their own variable names, or indentation, it's a red signal
for me.

## Giving hints during interview

The first couple interviews I conduct, I gave a lot of hints. I couldn't stand interviewees getting
stuck, or solving the problem using the less ideal method. But as time goes by I learn that it is
disrespectful to give the interviewee too much hint and it does no good at gauging the interviewee's
skill. There are different kinds of hints. Some are helpful to give, some are not.

### If the interviewee get stuck on an API

I would definitely give the hint. My team allows interviewee to look up any APIs in the interview.
We don't like testing candidates on their memory.

### If the interviewee misunderstood the question

I will also point out immediately. I feel it's part of my fault to not explaining the question
clearly.

### If the interviewee gets stuck because of a bug

I will wait a bit, letting him/her to debug it on their own. Everybody makes mistakes. As a software
developer, debugging skill is very important too. If I see the interviewee totally stuck and doesn't
know what to do, or blindly modifying the code and rerun to test it, I will point out the bug to
save both of us some time. At the same time, those hints will downgrade the feedback I give to
hiring manager.

### If the interviewee miss an special case

Hmm, it's hard to say whether I should give the hint. Usually I try to guide the interviewee to
tackle the problem in these steps:

1. understand the question
2. coming up with algorithm
3. code

Hopefully, the interviewee can see all the cases after step 2. Some cases can be simply handled by
an additional conditional statement. Some require using certain algorithms. For the later, it's
better to think about them before start coding, otherwise it might result in a complete rewrite of
the code. If the interviewee doesn't realize those cases, I will try my best to nudge him/her into
the direction.

### If the interviewee doesn't use the most ideal method

If they can solve the question, then I think it's not a very big deal. I mostly interview junior
position. As I said above, I do not emphasize too much on algorithm. Unless their method is going
take much longer to implement and them won't finish in time, I usually just let them do it.

## Giving feedback to hiring manager

This is the hardest part of the interview. It's clear that if I found someone better than me, then
it's an yes (ehh, because I am already hired). On the other end, if the interviewee is really bad,
it's easy to give a no. But most people lies in the "maybe" area. This
[guide](https://www.joelonsoftware.com/2006/10/25/the-guerrilla-guide-to-interviewing-version-30/)
gave me a lot of inspiration of determining if a "Maybe" is a hidden gem or just a no. Overall, my
guideline is: Do I want to work with him? If I need to refactor his/her code in 10 month and I feel
so bad about the code and cry, then it's a no.
