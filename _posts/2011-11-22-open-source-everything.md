---
layout: post
title: First Post
---

This is the first post of the blog.

{% highlight python %}

from pyres import ResQ
from pyres.worker import Worker
from pyres.job import Job

from statsd import statsd

# Stage used for namespacing.
STAGE = get_stage(APPROOT)

# Statsd Buckets.
VENMOQ_JOB_ENQUEUE = "venmoq.%s.job.enqueue" % STAGE
VENMOQ_JOB_COUNT = "venmoq.%s.job.count" % STAGE

REDIS_SERVER = "%s:%s" % (venmo_constants.REDIS_HOST,
                          venmo_constants.REDIS_PORT)

logger = logging.getLogger('lib.resq.init')
        
class VenmoWorker(Worker):
    """ Venmo wrapper around the Pyres Worker.

        Used for wrapping the handle exception emails, statsd counter, and
        dequeueing namespaced jobs.
    """

    ## Wrap error handler to send out emails.
    def failed(self):
        # TODO: should implement this in def _handle_job_exception(self, job)
        # after upgrading pyres, but the version of pyres we are using does not
        # support the _handle_job_exception hook
        exceptionType, exceptionValue, exceptionTraceback = sys.exc_info()
        handle_exception_in_queue_processor(exceptionValue)
        super(VenmoWorker, self).failed()

    ##############################
    ## Hooks provided by Pyres. ##
    ##############################
    def before_fork(self, job):
        """ Put stuff we want done before job here.
        """
        pass

    def after_fork(self, job):
        """ Put stuff we want done after job here.
        """
        # Track numbers of jobs in the queue.
        statsd.decr(VENMOQ_JOB_COUNT)

{% endhighlight %}

[Discuss this post on Hacker News](http://news.ycombinator.com/item?id=3267432)
