# Technical side of RLN

*This topic is a less strict version of specifications. If you want a more formal description, you can find specs in the [references](./references.md). Also, if you're unfamiliar with Shamir's Secret Sharing scheme, you can [read it here](./sss.md).*

___


![alt text](./images/rln-circuit.png)
<p align="center">
  <i>Under the hood: The <b>RLN</b> Circom Circuit</i>
</p>

**RLN** consists of three parts:
* User registration
* User interaction (signaling)
* User removal (slashing) - additional part

Well, let's discuss them.

## User registration
The first part of **RLN** is registration. There is nothing special in **RLN** registration; it's almost the same process as in other protocols/apps with anonymous environments: we need to create a Merkle Tree, and every participant must submit a `commitment` and place it in the Merkle Tree, and after that to interact with the app every participant will create a zkProof's, that they are a *member of the tree* (we use an *Incremental Merkle Tree*, as it more *GAS efficient*).

So, each member generates a secret key, denoted by \\(a_0\\). Identity commitment \\(q\\) is the hash (Poseidon) of the secret key: \\(q = Poseidon(a_0)\\).

**RLN** wouldn't work if there were no punishment for spam; that's why to become a member, a user has to register and provide something at stake. So, whoever has our \\(a_0\\) can "slash" us. 

The slight difference is that we must enable a *secret sharing* scheme (to split the `commitment` into parts). We need to come up with a polynomial. For simplicity we use linear polynomial (e.g. \\(f(x) = kx + b\\). Therefore, with two points, we can reconstruct the polynomial and recover the secret. 

Our polynomial will be: \\(A(x) = a_1 * x + a_0\\), where \\(a_1 = Poseidon(a_0, external\\_nullifier)\\).
The meaning of \\(external\\_nullifier\\) is described below.

## Signalling
Now that the user is registered, he wants to interact with the system. Imagine that the system is an *anonymous chat* and the interaction is the sending of messages. 
So, to send a message user have to come up with *share* - the point \\((x, y)\\) on her polynomial. 
We denote: \\(x = Poseidon(message), y = A(x)\\). 

Thus, if the same epoch user sends more than one message, their polynomial and, therefore, their secret (\\(a_0\\)) can be recovered.

Of course, we somehow must prove that our *share* = \\((x, y)\\) is valid (that this is really a point on our `polynomial = A(x)`), as well as we must prove other things are valid too, that's why we use zkSNARK. An explanation of the zk-circuits can be found in the next topic.

## Slashing
As it's been said, if a user sends more than one message, everyone else will be able to recover his secret, slash them and take their stake.

## Nullifiers
There are also \\(internal\\_nullifier\\) and \\(external\\_nullifier\\), which can be found in the **RLN** protocol/circuits.

\\(external\\_nullifier = Poseidon(epoch, rln\\_identifier)\\), where \\(rln\\_identifier\\) is a random finite field value, unique per RLN app.

The \\(external\\_nullifier\\) is required so that the user can securely use the same private key \\(a_0\\) across different **RLN** apps - in different applications (and in different eras) with the same secret key, the user will have different values ​​of the coefficient \\(a_1\\).

Now, imagine there are a lot of users sending messages, and after each received message, we need to check if any member can be slashed. To do this, we can use all combinations of received *shares* and try to recover the polynomial, but this is a naive and non-optimal approach. Suppose we have a mechanism that will tell us about the connection between a person and their messages while not revealing their identity. In that case, we can solve this without brute-forcing all possibilities by using a public \\(internal\\_nullifier = Poseidon(a_1)\\), so if a user sends more than one message, it will be immediately visible to everyone.

## Some important notes

Also, in our example (and [zk-chat](https://github.com/njofce/zk-chat) implementation), we use linear polynomial, but [SSS](sss.md) allows us to use various degree polynomials; therefore we can implement a protocol, where more than one signal (message) can be sent in per epoch. 

To learn more, check out the [specification](https://hackmd.io/7GR5Vi28Rz2EpEmLK0E0Aw?view); there are also [circuits](https://github.com/privacy-scaling-explorations/rln/tree/master/circuits) implemented for various degree polynomials too.
