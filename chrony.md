https://chrony.tuxfamily.org/faq.html#_is_code_chronyd_code_allowed_to_step_the_system_clock

Normally, it’s recommended to allow the step only in the first few updates, but in some cases (e.g. a computer without an RTC or virtual machine which can be suspended and resumed with an incorrect time) it may be necessary to allow the step on any clock update. The example above would change to

makestep 1 -1
