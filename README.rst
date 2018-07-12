Introduction
----------

This is a fork of the luigi package which was made to support scenarios where tasks are expected to fail and these tasks can tolerate these failures and should continue most of the time.

Details of customisation
----------
1. A new class ``luigi.SoftFailTask``
2. Changed logic in ``worker.py`` and ``scheduler.py`` for task processing: If a SoftFailTask fails, it is placed in the history as FAILED, but for the scheduler is processed as DONE.

Notes
-----

Tasks must still ensure that the complete() condition is handled. A SoftFailTask will therefore still catch BaseExceptions, write something trivial to the output Target (e.g. None), and then raise that exception above.

Example usage
-------------

::

   import luigi

   class simpleTask(luigi.Task, luigi.SoftFailTask):
       id = luigi.IntParameter()

       def output(self):
           return luigi.LocalTarget("test_{0}.tsv".format(self.id))

       def run(self):
           try:
               if self.id == 2:    
                   raise('This is arbirarily raised')
               open(self.output().path,'w').write('blah')
           except BaseException as e:
               open(self.output().path,'w').write('')
               raise e


       def requires(self):
           if self.id != 1:
               return (simpleTask(id=1))


   class aggTask(luigi.WrapperTask):    
       def requires(self):
           return [simpleTask(id=2),simpleTask(id=3),simpleTask(id=4)]

   class summaryTask(luigi.WrapperTask):    
       def requires(self):
           return [aggTask()]

   if __name__ == '__main__':
        luigi.build([summaryTask()], workers=1, local_scheduler=True)

Authors
-------

Luigi was built at `Spotify <https://www.spotify.com>`_, mainly by
`Erik Bernhardsson <https://github.com/erikbern>`_ and
`Elias Freider <https://github.com/freider>`_.
`Many other people <https://github.com/spotify/luigi/graphs/contributors>`_
have contributed since open sourcing in late 2012.
`Arash Rouhani <https://github.com/tarrasch>`_ is currently the chief
maintainer of Luigi.
