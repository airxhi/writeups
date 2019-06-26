# GoogleCTF _reality_
Solved by https://gist.github.com/elliptic-shiho/1d9d558e8de79a693410f1701fd39614?fbclid=IwAR1LXB8ZkZrAaDNzGMjYkq7Pu-M90KjisdeHLcv68ynCMKXAu6SqxS1pqPU

This is merely a technical writeup for understanding and clarification.

## Challenge Description
Shamir did the job for you, just connect to the challenge and get the flag.

Upon connecting to the given server, you're greeted with a base32 encoded and encrypted flag and 3 set's of `(x,f(x))` points called _coefficients_ which is wrong but whatever, and says you'll need 5 to solve the challenge.

## Walkthrough

So quite obviously we are working with Shamir's secret share (https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) which is a form of secret sharing that splits the secret into __n__ parts and requires a minimum of __k__ parts to reconstruct the secret.

Shamir's secret sharing is based around being able to constuct a (k-1)-degree polynomial with k points, with the secret being the constant coefficient in the equation and the other coefficients being randomly generated.
From the challenge description the polynomial is of degree 4 (a quartic), requiring 5 points to uniquely reconstruct.

What marks this implementation out as being incorrect is that we've been given real numbers and not integers. Working over a finite field is a requirement for Shamir's secret sharing to secure
as an infite ring gives us an ordering to how the polnomial is constructed and already gives us a vague idea of where the secret should lie.

An example set of point's would be

    x1 = 1.1523683371067123349656382131317521483572497...
    y1 = 785701515561796928936294283336388546237.76198...
    x2 = 1.9788681780306785745156989575455836997628595...
    y2 = 2800780348429075081385422902824585625983.9718...
    x3 = 1.1670934471504288389286699040597145166483859...
    y3 = 802707509246523948167720804790529379981.65200...
    
And so by interpolating the points using Lagrange interpolation we can see that the secret lie's in the `10e+37 - 10e+40` range. Now unfortunately for us this range is far too large to search through, so we need to use a smarter method that bruteforce to nab the secret.

We're going to represent our current information in a 3x6 matrix where the first row is each x<sub>n</sub> to the power of 4, and the next row is each x<sub>n</sub> to the power of 3 and so forth to 0, with the last row being the y<sub>n</sub> points.
    
    x1**4  x2**4  x3**4
    x1**3  x2**3  x3**3
    x1**2  x2**2  x3**2
    x1**1  x2**1  x3**1
    x1**0  x2**0  x3**0
    y1     y2     y3
    
Now, very interestingly, we can augment this matrix with a 6x6 identity matrix where each column is the coefficient for the 4-i'th x-term (and the last column being the coefficient of the y value).

    1  0  0  0  0  0  x1**4  x2**4  x3**4
    0  1  0  0  0  0  x1**3  x2**3  x3**3
    0  0  1  0  0  0  x1**2  x2**2  x3**2
    0  0  0  1  0  0  x1**1  x2**1  x3**1
    0  0  0  0  1  0  x1**0  x2**0  x3**0
    0  0  0  0  0  1  y1     y2     y3
