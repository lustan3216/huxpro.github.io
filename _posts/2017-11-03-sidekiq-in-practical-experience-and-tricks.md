---
layout:       post
title:        "9 ways to boost Sidekiq performance correctly in practical experiences"
subtitle:     "Lots of people miss crucial settings and use Sidekiq in poor efficiency way"
date:         2017-11-03 12:00:00
author:       "Luyi"
header-img:   "img/photo-sidekiq.jpeg"
header-mask:  0.3
catalog:      true
multilingual: true
tags:
    - Sidekiq
--- 
 

> I am preparing for the IELTS exam, so i’m trying to write articles in english.
> If you find any grammatical errors or technical issues, please do not hesitate to correct me in reply section below.

### Do not try to delete jobs 
In my case, there were more than a million jobs in my process, so query a job just like looking for a needle in a haystack.

Even though you might think your case not large quantity like me, or you might think Redis is doing so fast.

##### Here a trick could resolve this issue in efficient way.

The context is a advertisement which can be resumed or be paused, and both also can be scheduled in the future.

Bugs will produce by a situation you want to change the resume or pause schedule time multiply times.

Why?

If you set pause schedule at a month later, then want to change to two months.

The bug will happen on first one schedule you set will go first, and this situation could be really complicated as the resume and pause action do multi times.

So why don't you just check current job which the instance dose really need to execute when it perform ?!
   
```rb 
# Migrate start_at, resume_jid, end_at, pause_jid column on ad table.
# After any resume transition, it's will set the jid of last job in db.
 
  after_transition any => :resume do |ad, transition|
    pause_jid = PauseJob.set(wait_until: Time.current + 60.days).perform_later(ad.id).provider_job_id
  
    ad.update(resume_jid: nil, pause_jid: pause_jid)
  end   
      
# When job perform later, it will examine the provider_job_id does same with pause_jid the last time i set.
# If true then perform, and false then skip. 

  def perform(ad_id)
    ad = Ad.find_by_id(ad_id)   
      if ad && provider_job_id == ad.pause_jid
        ad.pause
      end   
  end
```
Just set xxx_jid column on table for check. It ought significantly reduce the bugs and more simpler than delete a job in large data.

### Using an initializer
Redis default number of databases is 16, 0 ~ 15.By default, Sidekiq use 0.

Jobs will conflict as the two or more project using Sidekiq at the same db.

Although they may not cause exception or bug, it could get slowly if unrelated job too much.

In order to prevent you forget to separate db in production just in case, you could setup db url after install Sidekiq immediately.   
 
It is important to note that to configure the location of Redis, you must define both the Sidekiq.configure_server and Sidekiq.configure_client blocks. To do this put the following code into config/initializers/sidekiq.rb.
  
```rb
  Sidekiq.configure_server do |config|
    config.redis = { url: 'redis://redis.example.com:7372/12' }
  end

  Sidekiq.configure_client do |config|
    config.redis = { url: 'redis://redis.example.com:7372/12' }
  end
```

[REFERENCE](https://github.com/mperham/sidekiq/wiki/Using-Redis)

### Do not use namespace in redis in order to separate rdb
Do not use namespace, not only using Sidekiq but also using Redis individual. 

Client side sometimes prefix key with namespace, e.g. "db1", so the key "db1" don't conflict with the key "db2"

The point is namespacing increases the size of every key by the size of the prefix and it will be expensive cost.

[REFERENCE](http://www.mikeperham.com/2015/09/24/storing-data-with-redis/)

### Pass id of instance rather than instance
Pass instance id e.g. `SomeWorker.perform_later(user_id)` rather than `SomeWorker.perform_later(user)`
```ruby
def perform(user_id)
  user = user.find(user_id)
  do_something(user) if user
end   
```
What the pattern benefit is that can recheck instance exist or not as it perform.

Supposing we execute `SomeWorker.set(wait_until: Time.current + 3.days).perform_later(user)` and the user had already deleted possible in this case.

Then the job will be set in retry zone which do the pointless loop.   

[REFERENCE](https://github.com/mperham/sidekiq/wiki/Best-Practices)

### Stringify params  
Do not put instance directly to be params, why?

* Put instance id can reduce the redis db size cost rather than an instance.

* Sidekiq client API uses `JSON.dump` to send the data to Redis.The Sidekiq server pulls that 
JSON data from Redis and uses `JSON.load` to convert the data back into Ruby types to pass to your perform method.
Don't pass symbols, named parameters or complex Ruby objects (like Date or Time!) as those will not survive the 
dump/load round trip correctly. 

### Need a callback when jobs complete or success in free way? 
Batch is pro function in Sidekiq, so if you want a callback you need to pay for pro version.


#### [sidekiq-batch](https://github.com/breamware/sidekiq-batch)
 
Here is a free way gem to use.

```rb

  class AdCost::CalculateAllJob < ApplicationJob 
  
    def perform
      Sidekiq::Queue.new('ad_cost').clear
  
      batch = Sidekiq::Batch.new
      batch.on(:complete, AdCost::CalculateAllCallback)
      batch.jobs do
        Ad.all.find_each do |ad|
          AdCost::CalculateOneJob.perform_later(ad.id)
        end
      end
    end
  end

  
  class AdCost::CalculateAllCallback
    
    # complete mean callback recall when all job do once, and not include retry or dead after 
  
    def on_complete(status, option={})
      # Need to keep those params, or its will not exceute     
      AdGroup::AdInfos.update
      Campaign::AdGroupInfos.update
    end
  
    # success mean callback recall when all job do successuly or when retry successuly.
  
    def on_success(status, option={})
      # Need to keep those params, or its will not exceute
      Manager::Budget.update
    end
  end
```

### Set pool: 25 in database.yml

By default, one sidekiq process creates 25 threads. If you didn't set pool: 25 in database.yml on production, it will cause lots bugs when large jobs.   

The author of Sidekiq also recommend not set the concurrency higher than 50.  

### Prevent dead Lock
Due to Sidekiq do jobs concurrency, its has lots chance to happen dead lock.

Rails provide a easy to way to prevent.

```rb
  ad.with_lock do
    ad.send(action)
    ad.do_someting
  end
```

### Set monintor
Mike Perham(Sidekiq builder) built this monintor, because he didn't like the existing tools that were available (e.g. monit, God and bluepill).
#### [inspeqtor](http://contribsys.com/inspeqtor/)

Here is Mike Perham recommendations: 
* Use [Upstart](https://github.com/mperham/sidekiq/tree/master/examples/upstart) or [Systemd](https://github.com/mperham/sidekiq/tree/master/examples/systemd) to start/stop Sidekiq. 
This ensures if the Ruby VM crashes, the process will respawn immediately.
* Use Inspeqtor to monitor the CPU and memory usage and restart Sidekiq if necessary.


[REFERENCE](https://github.com/mperham/sidekiq/wiki/Monitoring)




