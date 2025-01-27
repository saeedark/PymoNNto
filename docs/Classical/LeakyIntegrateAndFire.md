# Leaky Integrate and Fire Neuron PymoNNto Implementation

The following code creates a network with 800 excitatory neurons, 200 inhibitory neurons and all the connections between them.
The neurons are simple leaky integrate and fire neurons which receive inhibitory and excitatory input.

```python

from PymoNNto import *


class LIF_main(Behavior):

    def initialize(self, n):
        self.set_parameters_as_variables(n)
        n.v = n.vector('uniform') * n.v_rest
        n.fired = n.vector() > 0
        n.dt = 0.1

    def iteration(self, n):
        n.v += ((n.v_rest - n.v) + n.I) * n.dt

        n.fired = n.v > n.v_threshold
        if np.sum(n.fired) > 0:
            n.v[n.fired] = n.v_reset


class LIF_input(Behavior):

    def initialize(self, n):
        for s in n.synapses(afferent):
            s.W = s.matrix('uniform')

        n.I = n.vector()

    def iteration(self, n):

        n.I = 90 * n.vector('uniform')

        for s in n.synapses(afferent, 'GLUTAMATE'):
            n.I += np.sum(s.W[:, s.src.fired], axis=1)

        for s in n.synapses(afferent, 'GABA'):
            n.I -= np.sum(s.W[:, s.src.fired], axis=1)


My_Network = Network()

N_e = NeuronGroup(net=My_Network, tag='excitatory_neurons', size=get_squared_dim(800), behavior={
    1: LIF_main(v_rest=-65, v_reset=-80, v_threshold=-10),
    2: LIF_input(),
    9: Recorder(['v', 'fired'], tag='my_recorder')
})

N_i = NeuronGroup(net=My_Network, tag='inhibitory_neurons', size=get_squared_dim(200), behavior={
    1: LIF_main(v_rest=-65, v_reset=-80, v_threshold=-30),
    2: LIF_input(),
    9: Recorder(['v', 'fired'], tag='my_recorder')
})

SynapseGroup(net=My_Network, src=N_e, dst=N_e, tag='GLUTAMATE')
SynapseGroup(net=My_Network, src=N_e, dst=N_i, tag='GLUTAMATE')
SynapseGroup(net=My_Network, src=N_i, dst=N_e, tag='GABA')
SynapseGroup(net=My_Network, src=N_i, dst=N_i, tag='GABA')

My_Network.initialize()

My_Network.simulate_iterations(1000, measure_block_time=True)

import matplotlib.pyplot as plt

plt.plot(My_Network['v', 0])
plt.show()

plt.imshow(My_Network['fired', 0, 'np'].transpose(), cmap='gray', aspect='auto')
plt.show()

# from PymoNNto.Exploration.Network_UI import *
# my_UI_modules = get_default_UI_modules(['fired', 'v', 'u'], ['W'])
# Network_UI(My_Network, modules=my_UI_modules, label='My_Network_UI', group_display_count=2).show()

```



















