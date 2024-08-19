Result
======
In this section, we will show the reproduction of the work [1]_ first. And then we will use the same evaluation
metrics to compare the performance of the BIBD and SPG encoding schemes with other exact gradient coding.


Reproduction  
------------
First we will setup experiments to test out the abilities of uncoded(naive) scheme, FRC scheme, CRC scheme, and
ignore straggler scheme. This horizontal comparison mainly aims to prove that encoded distributed learning will
have faster average iteration training time and perform as good as the uncoded version.

Below are the two comparison figures. The first one tells us the speed of training per iteration after 100 total
iteration training on Amazon dataset. We see that both our reproduction and original work have shown that encoded
version will have faster training time per iteration. This suggests that by adopting gradient code, we are able
to achieve faster training per iteration. This could be possible that when certain worker becomes straggler in 
the training, the encoded cluster can instantly drop the straggler and recover full gradient at the master node, 
while uncoded cluster could only be kept waiting. This is especially more obvious when we have a larger cluster.

In the second figure, we are able to see that both FRC and CRC reaches 0.88 AUC in the end (left and right), which
is similar to the original work. However, since ignore version will discard the straggler in the real time, we 
cannot guarantee to reproduce the exact final result as shown in the original work. In fact, throughout our tests,
we failed to have one test of ignore version with final AUC value at around 0.84. But one can verify, that certain
inflection points on the curve corresponding to the timestamp are around the same between the two images as for
FRC and CRC coded version.

    .. image:: result/reproduction_avgtime.png
        :alt: reproduction_avgtime
        :width: 400px
        :height: 180px
        :align: center

    .. image:: result/reproduction_auc.png
        :alt: reproduction_auc
        :width: 400px
        :height: 180px
        :align: center

In summary, it is believed that this project has similar ground performance with the original work. And we will
build more tests and comparisons on top of it.


BIBD vs SPG vs Uncoded
----------------------
This section compares the full evalutation metrics over BIBD, SPG and uncoded scenarios.
This set of test is run agains the Amazon dataset for 100 iteration per round, in total of 42 rounds.
Note that whenever there's a bad data due to AWS server down and totally unrecoverable, it is represented as 0 in chart.

The first figure demonstrates the average training time per iteration, where at first three versions have similar
average iteration time and the BIBD outperforms a little from round 10 to 20. However, as the round increases and much
compuatational burden is put on to the cluster, we can see that the approximate gradient code BIBD and SPG maintain
a reasonable performance as before, while uncoded cluster suffers from the high latency each round.
This gives us huge confidence on the gradient code when the network congestion or abnormal latency happens.

    .. image:: result/bibd_spg_avgtime.png
        :alt: reproduction_avgtime
        :width: 400px
        :height: 200px
        :align: center

The second figure suggests that both BIBD and SPG gradient code's resilience to straggling are coming from the expense
at model performance. We can see that BIBD and SPG's model AUC fluctuate at each round but overall stay near the 
uncoded baseline. No outperforming is observed in this test.

    .. image:: result/bibd_spg_auc.png
        :alt: reproduction_auc
        :width: 400px
        :height: 200px
        :align: center

The third figure shows that for the Amazon dataset, the approximate gradient code like BIBD and SPG do not significantly
affect the accuracy of the model. It is partially due to the extreme unbalanced dataset. But it is worth stating that 
gradient code maybe preferred in this case to trade model accuracy to the training time.

    .. image:: result/bibd_spg_acc.png
        :alt: reproduction_auc
        :width: 400px
        :height: 200px
        :align: center


Conclusion
----------
In this project, we have experimented with various gradient coding ideas on Amazon EC2 instances. 
This is a complex trade-off space between model sizes, number of workers, worker configurations, and final performances. 
We are glad to see that the proposed SPG coding from our lab achieves similar effect to the BIBD encoding, while SPG 
has more flexibility than BIBD. Moreover, this universal test pipeline would be helpful for any determined researcher
to try out their ideas with standardized and also real-life machine learning training application.
The main benefit of the gradient code is fault tolerance, as the big data is becoming common, we are hoping to see more
creative ideas emerging from the coding theory to practical applications.

.. References
.. ..........

.. [1] Tandon, R., Lei, Q., Dimakis, A. G., & Karampatziakis, N. (2016). Gradient coding. arXiv preprint 
   arXiv:1612.03301.