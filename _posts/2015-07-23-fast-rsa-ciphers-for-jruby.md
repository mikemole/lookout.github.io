---
layout: post
title: "Faster RSA Ciphers on JRuby"
tags:
- java
- jruby
- jruby-openssl
- ssl
---

When most people think about Lookout, they rarely consider more software than
what is available in the Android or iOS App Stores. But truth be told, the vast
majority of what powers Lookout, and in fact makes us Lookout, is a very large
set of backend web and data services offloading a tremendous amount of work
from mobile clients themselves. The platform powering a non-trivial amount of
this backend infrastructure is [JRuby](http://jruby.org).

The JRuby runtime is compelling for a number of reasons, one of the most often
touted is performance. Like many users we've generally seen good performance
from JRuby, so imagine our surprise when we found ponderously slow performance
in some of our OpenSSL operations.

Despite generally positive Ruby performance in JRuby, overall performance is
still tied to the capabilities of the JVM itself. In the case of OpenSSL
operations, what's really being compared is the performance of Java's SSL
library versus the [openssl's](http://openssl.org/) C library. Unfortunately,
for RSA-specific operations, Java has some slower math operations under the
covers.  For example, here's some benchmark code using RSA ciphers for
signing/signature verification/encryption/decription on MRI versus JRuby
1.7.21:


    $ ruby2.1 -v benchmark.rb
    ruby 2.1.2p95 (2014-05-08) [x86_64-linux-gnu]
                            user     system      total        real
    sign                9.710000   0.000000   9.710000 (  9.706266)
    verify              0.150000   0.000000   0.150000 (  0.158150)
    encrypt             0.170000   0.000000   0.170000 (  0.167504)
    decrypt             9.720000   0.000000   9.720000 (  9.716763)

    $ ruby -I benchmark.rb
    jruby 1.7.21 (1.9.3p551) 2015-07-07 a741a82 on OpenJDK 64-Bit Server VM
    1.7.0_79-b14 +jit [linux-amd64]

                            user     system      total        real
    sign               45.600000   0.230000  45.830000 ( 45.451000)
    verify              0.950000   0.000000   0.950000 (  0.810000)
    encrypt             0.790000   0.060000   0.850000 (  0.815000)
    decrypt            45.280000   0.080000  45.360000 ( 45.257000)

Signing and decryption operations in JRuby are **5x** slower than their
counter-parts in MRI!


Fortunately, this is a known problem and our colleagues at
[Square](http://squareup.com) have [published jnagmp and
bouncycastle-rsa](https://corner.squareup.com/2014/02/faster-rsa-jnagmp.html)
which provides some faster math calls to the JVM, significantly improving the
performance of RSA operations on the JVM.


Today we're releasing a small gem called
**[fast-rsa-engine](https://github.com/lookout/fast-rsa-engine)** which we've
written to incorporate Square's "bouncycastle-rsa" into the
[jruby-openssl](https://github.com/jruby/jruby-openssl) library.

Revisiting that same benchmark from above with fast-rsa-engine loaded into the
JRuby runtime:

    $ ruby -v benchmark.rb
    jruby 1.7.21 (1.9.3p551) 2015-07-07 a741a82 on OpenJDK 64-Bit Server VM
    1.7.0_79-b14 +jit [linux-amd64]
                            user     system      total        real
    sign               11.160000   0.000000  11.160000 ( 11.055000)
    verify              0.450000   0.010000   0.460000 (  0.324000)
    encrypt             0.420000   0.050000   0.470000 (  0.343000)
    decrypt            11.050000   0.080000  11.130000 ( 10.972000)


Instead of **5x** slower, it's now only 1.2x slower, which is a big speed up!


These improvements exist as a stand-alone gem currently, and may in the future
be incorporated into jruby-openssl. For now however, you can install the gem
from [rubygems.org](http://rubygems.org/gems/fast-rsa-engine) with `gem install
fast-rsa-engine`


If you're interested in talking about JRuby, OpenSSL or fun on the JVM, come
talk to us [at JRubyConf EU 2015!](/2015/07/lookout-at-jrubyconfeu/)
