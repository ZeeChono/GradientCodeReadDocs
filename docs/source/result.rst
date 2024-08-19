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


Comparison
----------


.. References
.. ..........

.. [1] Tandon, R., Lei, Q., Dimakis, A. G., & Karampatziakis, N. (2016). Gradient coding. arXiv preprint 
   arXiv:1612.03301.