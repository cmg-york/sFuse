# Overview

The following is the HTN SHOP2 sample specification accompanying our conference paper submission on model-driven implementation of security requirements using patterns. [The listing](https://github.com/genericaylo/submission/blob/main/src/Example.lisp) can be downloaded as a LISP file from the repository. It contains: (a) workflow patterns for securely transmitting information between two actors using various communication channels and utilizing cryptographic primitives, (b) example attack trees for confidentiality, authentication and integrity. In later sections, we use these assets to reason with various security requirements and vulnerability assumptions. 

Although the presented models here is much more complex than the ones in the paper (e.g. second order LISP-like terms are extensively used), they are constructed following the same principles. The formalization can be seen as one of the many possible that can be devised for solving the same problem domain (securely transferring documents between two parties) and is given as an example of what is possible rather than an authoritative model for the domain. The formalization of choice is a result of balancing between expressiveness, exact reasoning needs (e.g., do we need to reason about many different channels?), and computational efficiency and requires substantial validation effort with domain experts. These tasks are central for our future research agenda.

[The listing](https://github.com/genericaylo/submission/blob/main/src/Example.lisp) can be compiled and used as-is by the SHOP2 planner. Installation instructions are provided below.


# The Model

## Key aspects
The specification focuses on the exchange of information between two parties making certain assumptions on the devices and media in which the information is stored, and the computational devices that the participants use. The possibilities are by no means meant to be exhaustive but rather to provide an illustration of the modeling and reasoning possibilities available through the framework. 

<!--
Note that the entire domain specification, except for the attack tree axioms and the final method, is really an HTN description of an large and complex AND/OR goal decomposition, where methods are used to implement intermediate AND and OR decomposed nodes. Thus, the entire specification can be visualized using iStar 2.0 constructs. 
-->

Note that the entire information exchange security decomposition pattern, together with its attack trees is given in the form of a textual HTN specification. Its diagrammatic visualization using iStar T constructs is possible but has been omitted here for simplicity. 



## Computational Devices
Participants in the workflows are assumed to have access to the following devices:

* A PC or other computer connected to the internet. The PC is the only devices where cryptographic functions can be executed.
* A mobile phone, capable of sending text messages with file attachments. Actors can connect their phone to their PC to exchange files. 
* A printer and a scanner (with no handwriting recognition). 
* USB drives or other portable digital storage medium

## Formats and Media

Information is at any point in time available with an actor. A special predicate is used to signify this:
```lisp
(has ?agent ?info ?medium ?format)
```
While ``` ?agent ``` and ```?info``` can be anyone/anything, ```?medium``` and ```?format``` are restricted as follows:

Possibilities for medium:

* Local-drive: meaning the hard-drive of the PC
* shared-folder: implies that the file exists also in the cloud and accessible for anyone with access to the cloud account.
* mailbox: the inbox or outbox of an agent’s email account, which is assumed to reside both on a server and as a local copy.
* phone: any part of the agent’s smartphone (e.g. text messages, file-system or email app), such that hacking the phone implies full access to that location.
* USB: shorthand for any portable digital storage device, including e.g. CDs, DVDs or other media.
* Physical: any physical location in which an agent has access: e.g., their pocket or their desk. 

Possibilities for format:

* digital-file
* paper which can be either printed (thus, OCR-able) or handwritten.

Information can be scanned, typed-up and copied from one machine to another. Printing information is omitted for simplicity; anything digitally available can be assumed to printed as well. We generally assume that users and/or the technology they use will perform actions that inadvertently expand the attack surface by copying the information from one medium to the other. For example, the following assumptions:

* Information in shared-drive will eventually sync to the local-drive.
* Information in the (e-)mailbox (e.g. an attachment), will be downloaded to local-drive.
.. mean that compromising a user's PC automatically implies that information in their server-side mailboxes and shared drive is also vulnerable through user syncronization actions or, e.g., login monitoring and credentials theft.

## Transferring information
Information can be transferred in any of the following ways:

* Email.
* SMS, i.e., text messaging, which is assumed to also support attachments.
* Phone-call oral exchange, whereby one actor is calling another and offers the information verbally. The other actor is assumed to jot down the information manually in their physical space. 
* In person exchange, whereby one actor visits the physical space of another actor and delivers the information in paper or digital format (e.g. USB key).
* In person oral exchange, like phone-call oral exchange but requiring a visit to another person's space.

## Encryption and Decryption Methods

Encryption can be _symmetric_ or _asymmetric_. In the former case the participants exchange a shared key and use their document viewer or word processor to protect/encrypt the document with the key. As such, no extra software is required. However, the shared key needs to be exchanged securely. 

For asymmetric encryption public/private key pairs need to be generated and the public parts exchanged in a way that is not necessarily secured. We however assume that the standard versions of document viewers and processors do not trivially support public key cryptography. As such, specialized software needs to be installed for such, such as for example [OpenPGP](https://www.openpgp.org/).

## Digital Signatures
Digital signing is asymmetric through the use of third-party software such as [OpenPGP](https://www.openpgp.org/). It is further assumed that signing and encryption take place independently and in this order when both are needed. Uncompromised signing guarantees authentication, protection against tampering and non-repudiation.

The model does not yet support message authentication codes (MAC) for symmetric encryption.

## Key management
Key management is a step that generally precedes encryption and/or signing, and involves key generation and sharing. The sharing of keys follows the same methods as the sharing of any other information. However, advanced key establishment protocols (e.g., Diffie-Hellman) are not currently included as methods, due to being unrelated to the context and examples of use we are considering here. Thus, shared key exchange, which is sensitive information, must take place using a channel that is assumed to be secure (e.g., in person exchange, a phone call or other method) depending on the *vulnerability assumptions* in effect. 

## Attack Trees and Security Requirements
Attack trees are simple axioms connecting a high-level characterization of the attack (the negation of a security requirement) with a logical formula describing conditions under which the attack is accomplished, hence the security requirement successfully breached. Notice that the components of the formula appear in any of: effects and preconditions of operations and/or methods, vulnerability assumptions or domain assumptions. 


Each security requirement is connected with a conjunction of the negation of the attack tree that can compromise it, and, potentially, a formula that describes other conditions for the requirement to be fulfilled; both conjuncts are treated as necessary conditions for the fulfillment of the security requirement. 


The following attack trees are introduced:

* **Con** ``` (ConBreached ?sender ?recipient ?info) <= ...``` followed by various combinations of effects and assumptions that make such breach possible. The predicate means that a piece of ```?info``` transmitted from ```?sender``` to ```?receiver``` was read by an unauthorized third party.
* **Int** ``` (IntBreached ?sender ?recipient ?info) <= ...``` followed by various combinations of effects and assumptions that make such breach possible. The predicate means that a piece of ```?info``` transmitted from ```?sender``` to ```?receiver``` was altered by an unauthorized third party. The patterns do not cover hash functions and message authentication codes, hence, in its current state, the axiom simply blocks transmission from compromised channels. The conjunct ```(not (authenticated ?recipient ?sender ?info))``` can be added to trigger integrity protection via digital signatures.
* **Auth** ``` (AuthSucc ?sender ?recipient ?info) <= (authenticated ?recipient ?sender ?info) ``` meaning that authentication is successful for ```?info``` being transmitted from ```?sender``` to ```?recipient```, if the ```?recipient``` was able to authenticate that the ```?info``` was indeed of the ```?recipient```. In addition ```(AuthBreached ?sender ?recipient ?info) <= ``` describes conditions under which the private key of the signer is compromised, allowing the attacker to impersonate as the signer, breaching authentication. 
* **NonRep** ```(NonRepSucc ?pronouncer ?info ?recipient) <= (cannot-repudiate ?pronouncer ?info ?recipient)``` meaning that ```?pronouncer``` cannot repudiate that  ```?info``` never originated by them if in the right-hand-side effect is true. The latter is the effect of the recepient's verification action. Note that if the signing key is stolen, this is an attack on the sender and not the recipient, who can still claim non-repudiation.

A few remarks here:

* The attack tree "heads" (the LHS of the implications) describe a breach within the context of a transmission of information from a sender to a receiver, since different requirements may be posed for different such instances.
* The heads can be negative (describe a breach via an attack tree, to be negated when posing the requirement, positive (describe conditions required for the security qualities), or both.
* The axioms are provisional and subject to future dedicated analysis. Our goal here is, again, to showcase the modeling technique rather than offer authoritative definitions.
* Additional axioms may be needed for various security or privacy requirements and complex formulations thereof including non-disclosure, redundancy, retention, or disposal etc.

## Domain and Security Requirements
As mentioned in the paper, a special action is added in the domain specification for the purpose of enforcing its precondition. This is due to the fact that SHOP2’s problem specification is written in the form of a top level method rather than a goal state. Thus, a method we call ```final``` is introduced and security and other domain requirements are added as preconditions. Successful fulfillment of these preconditions allows the planner to execute “done”, a dummy final action.

## Problem Specification

### Threat Model
The domain and vulnerability assumptions (the thread model) are part of the problem specification and specifically the description of the _initial state_. Thus:

```lisp
	(defproblem problem1 sec
		(
		; +++++++++++++++++++++++++++++++++++++++++
		; + D O M A I N    A S S U M P T I O N S  +
		; +++++++++++++++++++++++++++++++++++++++++
	   
		(has contractor order local-drive digital-file)
		(has supplier invoice local-drive digital-file)
	
		(allow email)
		(allow phonecall)

		(has contractor supplier email-address)
		(has supplier contractor email-address)
		(has contractor supplier phone-number)
		(has supplier contractor phone-number)
	
		; +++++++++++++++++++++++++++++++++
		; +   V U L N E R A B I L I T Y   +     
		; +     A S S U M P T I O N S     +
		; +++++++++++++++++++++++++++++++++
		(is-compromised network)
		(is-compromised contractor mailbox)

	) 
	
	;...  main problem below
```

### Main Problem Definition
Problem definition is based on the specification of the top-level method (in our case ```transmit-information```) preceded by a ```manage-keys``` call and followed by ```final```, which must be achieved in any successful plan. Obviously, the three constituent methods can be further abstracted into a top level method.

```lisp

	; ... initial state above
	
	(:ordered
		(manage-keys doctor patient)
		(transmit-information doctor patient proof-of-visit)
		(final)
	)
```

Alternatively we can define a domain specific method:
```lisp
    (:method 
	(sendInvoice ?sender ?recipient) ;name
	() ; precondition
	(:ordered
		(manage-keys ?sender ?recipient)
		(transmit-information ?sender ?recipient invoice)
		(final)
	)
    )
```

... and have the main method use that instead:

```lisp
	; ... initial state above
	(:ordered
		(sendInvoice supplier contractor)
	)
```

# Relationship to Goal Models

The image below shows how the various visual elements developed in iStar-T and STS-ml are translated into chunks of the HTN specs. The actual transmission of the invoice (a domain specific requirement) is matched with security decomposition pattern _Transmit Document [Sender, Recipient, Document)_ which is instantiated into _Transmit Document [supplier, customer, invoice)_ according to the STS-ml social model. Instantiation implies adoption of the entire HTN specification that maps to the generic decomposition that is part of the pattern (box (1)). It is also accompanied with instantiation of the corresponding initialization element, which contains (a) the security requirements marked in the STS-ml model, and (b) domain and vulnerability assumptions relating to the problem at hand. In HTN terms, the former are injected into the HTN domain specification (box (3)) and the latter into the HTN problem specification (box (2)). Note also that the instantiated call to the root of the generic decomposition    resides into the problem specification as the root HTN task ```(transmit-document supplier contractor invoice)```.


<img src="https://github.com/genericaylo/submission/blob/main/img/translation.png?raw=true" alt="Description of relationship between goal diagrams and resulting HTN spec chunks" width="450"/>

# Running 
To identify plans, SHOP2 requires running 
```lisp
(find-plans 'problem1 :verbose :plans :optimize-cost t)
```

Please see below for rough installation instructions.

## Running Example

Let us now explore the example of the interaction between the supplier and the contractor. Recall that the interaction is that the contractor places an order to the supplier, who, in turn issues an invoice to be sent to the contractor. We focus on the transmission of the invoice from the supplier to the contractor. The output of all trials can be found here [here](https://github.com/genericaylo/submission/blob/main/src/out.txt).

### Case 1 – No security requirements and no vulnerability assumptions.

We start with the scenario in which no security requirements are given. Thus in the final method we have:

```lisp
    (:method 
    (final) 
	(	; ++++++++++++++++++++++++++++++++++++++++++
		; + D O M A I N    R E Q U I R E M E N T S +
		; ++++++++++++++++++++++++++++++++++++++++++
		
		(can-read contractor invoice)
		
	   
		; ++++++++++++++++++++++++++++++++++++++++++++++
		; + S E C U R I T Y    R E Q U I R E M E N T S +
		; ++++++++++++++++++++++++++++++++++++++++++++++

		;empty
	)
		(done))
    )
```

... and the problem definition is ...

```lisp
(defproblem problem1 sec
	(
		; +++++++++++++++++++++++++++++++++++++++++
		; + D O M A I N    A S S U M P T I O N S  +
		; +++++++++++++++++++++++++++++++++++++++++
	   
		(has supplier invoice local-drive digital-file)

		(allow email)
		(allow phonecall)

		(has contractor supplier email-address)
		(has supplier contractor email-address)
		(has contractor supplier phone-number)
		(has supplier contractor phone-number)

   
		; +++++++++++++++++++++++++++++++++
		; +   V U L N E R A B I L I T Y   +     
		; +     A S S U M P T I O N S     +
		; +++++++++++++++++++++++++++++++++
		
		;empty
	)
	
	(:ordered
		(manage-keys supplier contractor)
		(transmit-information supplier contractor invoice)
		(final)
	)
	
) ; end of problem specification
```

In such case, the planner will not be constrained in any way to produce the cheapest possible plan, which may involve the simple transfer of the document from sender to recipient in plaintext, thus plan:

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1  -178.0  -178.0        374       6605     0.063      0.114

Plans:
(((!NA) (!NA) (!NA) (!NA) (!NA) (!NA) (!EMAIL SUPPLIER CONTRACTOR INVOICE)
  (!NA) (!NA) (!NA) (!DONE)))
```

When looking at the plan we discard all the ``` (!NA) ``` actions, which simply signify optional actions (e.g. key creation and exchange, encryption, etc.) that were not chosen. What remains is a simple email action of the invoice from the supplier to the contractor, without any security steps. Notice also the CPU time and inferences it takes to generate the plan, to compare with the examples that follow.


### Case 2 – Invoice confidential.

Let us now assume that we specify a confidentiality requirement: the contractor wants to be able to ensure that the invoice has not been read by unauthorized parties. We state this requirement through a negated attack:

```lisp
		; ++++++++++++++++++++++++++++++++++++++++++++++
		; + S E C U R I T Y    R E Q U I R E M E N T S +
		; ++++++++++++++++++++++++++++++++++++++++++++++

		(not (ConBreached supplier contractor invoice))
```

However, we do not add any vulnerability assumptions. The planner will return the exact same plan:

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1  -178.0  -178.0        374       6911     0.078      0.134

Plans:
(((!NA) (!NA) (!NA) (!NA) (!NA) (!NA) (!EMAIL SUPPLIER CONTRACTOR INVOICE)
  (!NA) (!NA) (!NA) (!DONE)))
```

That is, there will be no security steps when the attackers are not assumed to engage in any attack. If we do assume we protect against certain attacks, such as for example, compromised mailboxes and networks then we need to add the corresponding vulnerability assumptions:


```lisp
		; +++++++++++++++++++++++++++++++++
		; +   V U L N E R A B I L I T Y   +     
		; +     A S S U M P T I O N S     +
		; +++++++++++++++++++++++++++++++++

		(is-compromised network)
		(is-compromised contractor mailbox)
```

Then the planner will resort to encryption:

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1   265.0   265.0      35123     841859     6.641     10.712

Plans:
(((!NA) (!NA) (!GENERATE-SYMMETRIC-KEY SUPPLIER CONTRACTOR) (!NA)
  (!BY-PHONECALL-EXCHANGE SUPPLIER CONTRACTOR (KEY SUPPLIER CONTRACTOR SHARED))
  (!TYPE-UP CONTRACTOR (KEY SUPPLIER CONTRACTOR SHARED)) (!NA) (!NA)
  (!SYMMETRIC-ENCRYPT SUPPLIER CONTRACTOR INVOICE
   (KEY SUPPLIER CONTRACTOR SHARED))
  (!EMAIL SUPPLIER CONTRACTOR
   (ENCRYPTED INVOICE (KEY SUPPLIER CONTRACTOR SHARED)))
  (!SYMMETRIC-DECRYPT CONTRACTOR SUPPLIER INVOICE) (!NA) (!NA) (!DONE)))
``` 

The plan says that the supplier now needs to generate a key, call the contractor to share the key and then symmetrically encrypt the document before sending it by email. In practice, this may means that the PDF or DOC is protected with a password within the corresponding document editing or word processing tools; and the password is exchanged via a channel for which no vulnerability has been assumed. The planner avoids asymmetric encryption which is more expensive as it requires (it is assumed here) installation of specialized software.

### Case 3 – Contractor wants to authenticate the invoice.

Let us now assume that we specify an authenticity requirement: the contractor wants to ensure that the person purporting to be the supplier is indeed the supplier. We then consider the following security requirements while keeping all else as-is:

```lisp
		; ++++++++++++++++++++++++++++++++++++++++++++++
		; + S E C U R I T Y    R E Q U I R E M E N T S +
		; ++++++++++++++++++++++++++++++++++++++++++++++

		(and	(AuthSucc supplier contractor invoice)
				(not (AuthBreached supplier contractor invoice))
		)
```

The plan that will return now is different:

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1  2544.0  2544.0      10300     229443     1.984      3.212

Plans:
(((!INSTALL-ENCRYPTION-SOFTWARE SUPPLIER)
  (!INSTALL-ENCRYPTION-SOFTWARE CONTRACTOR)
  (!GENERATE-ASYMMETRIC-KEYS SUPPLIER) (!NA)
  (!EMAIL SUPPLIER CONTRACTOR (KEY SUPPLIER PUBLIC)) (!NA) (!NA) (!NA) (!NA)
  (!ASYMMETRIC-SIGN SUPPLIER INVOICE (KEY SUPPLIER PRIVATE)) (!NA)
  (!EMAIL SUPPLIER CONTRACTOR (SIGNED INVOICE (KEY SUPPLIER PRIVATE))) (!NA)
  (!ASYMMETRIC-VERIFY CONTRACTOR SUPPLIER
   (SIGNED INVOICE (KEY SUPPLIER PRIVATE)) (KEY SUPPLIER PUBLIC))
  (!NA) (!DONE)))
```

In this case the supplier will digitally sign the document prior to sending it to the contractor. However, encryption software needs to be installed (e.g. [OpenPGP](https://www.openpgp.org/software/kleopatra/)) and the supplier needs to generate a public-private key pair and share the public one. It does not matter that the public key goes through a compromised medium (email).


### Case 4 – Authenticate _and_ encrypt the invoice.

Let us further assume that both authentication and encryption are needed:

```lisp
		; ++++++++++++++++++++++++++++++++++++++++++++++
		; + S E C U R I T Y    R E Q U I R E M E N T S +
		; ++++++++++++++++++++++++++++++++++++++++++++++

		(not (ConBreached supplier contractor invoice))
		(AuthSuccessful supplier contractor invoice)
```

Then we will get the following plan:

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1  2788.0  2788.0      38166     920827     7.906     10.848

Plans:
(((!INSTALL-ENCRYPTION-SOFTWARE SUPPLIER)
  (!INSTALL-ENCRYPTION-SOFTWARE CONTRACTOR)
  (!GENERATE-ASYMMETRIC-KEYS SUPPLIER) (!NA)
  (!EMAIL SUPPLIER CONTRACTOR (KEY SUPPLIER PUBLIC)) (!NA) (!NA)
  (!GENERATE-SYMMETRIC-KEY SUPPLIER CONTRACTOR) (!NA)
  (!BY-PHONECALL-EXCHANGE SUPPLIER CONTRACTOR (KEY SUPPLIER CONTRACTOR SHARED))
  (!TYPE-UP CONTRACTOR (KEY SUPPLIER CONTRACTOR SHARED)) (!NA)
  (!ASYMMETRIC-SIGN SUPPLIER INVOICE (KEY SUPPLIER PRIVATE))
  (!SYMMETRIC-ENCRYPT-SIGNED SUPPLIER (SIGNED INVOICE (KEY SUPPLIER PRIVATE))
   (KEY SUPPLIER CONTRACTOR SHARED))
  (!EMAIL SUPPLIER CONTRACTOR
   (ENCRYPTED (SIGNED INVOICE (KEY SUPPLIER PRIVATE))
    (KEY SUPPLIER CONTRACTOR SHARED)))
  (!SYMMETRIC-DECRYPT-SIGNED CONTRACTOR SUPPLIER INVOICE)
  (!ASYMMETRIC-VERIFY CONTRACTOR SUPPLIER
   (SIGNED INVOICE (KEY SUPPLIER PRIVATE)) (KEY SUPPLIER PUBLIC))
  (!NA) (!DONE)))
```

The above plan suggests that the document will be signed using a public key exchanged by email and then symmetrically encrypted, with key exchanged over a phone call, before it is sent over by email for decryption and verification.

One may doubt, however, that this is the most convenient way of doing it. Indeed, the specification does not have a fine-tuned cost scheme at this point that would recognize that, since asymmetric encryption software has been installed for the signatures, encryption may as well be asymmetric, saving the phone call for the shared key exchange. Even then, though, the analysts can simply artificially disallow symmetric encryption, e.g. through an ad-hoc precondition or assumptions such as:

```lisp
		; ++++++++++++++++++++++++++++++++++++++++++++++
		; + S E C U R I T Y    R E Q U I R E M E N T S +
		; ++++++++++++++++++++++++++++++++++++++++++++++

		... [as above]
		(not (has supplier (key supplier contractor shared) local-drive digital-file)
```
So now, we demand that at no point is there a shared key generated. The planner will respond as follows:

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1  3526.0  3526.0     215114    5339915    49.672     65.178

Plans:
(((!INSTALL-ENCRYPTION-SOFTWARE SUPPLIER)
  (!INSTALL-ENCRYPTION-SOFTWARE CONTRACTOR)
  (!GENERATE-ASYMMETRIC-KEYS SUPPLIER) (!NA)
  (!EMAIL SUPPLIER CONTRACTOR (KEY SUPPLIER PUBLIC)) (!NA) (!NA) (!NA)
  (!GENERATE-ASYMMETRIC-KEYS CONTRACTOR) (!NA)
  (!EMAIL CONTRACTOR SUPPLIER (KEY CONTRACTOR PUBLIC)) (!NA) (!NA) (!NA)
  (!ASYMMETRIC-SIGN SUPPLIER INVOICE (KEY SUPPLIER PRIVATE))
  (!ASYMMETRIC-ENCRYPT SUPPLIER CONTRACTOR
   (SIGNED INVOICE (KEY SUPPLIER PRIVATE)) (KEY CONTRACTOR PUBLIC))
  (!EMAIL SUPPLIER CONTRACTOR
   (ENCRYPTED (SIGNED INVOICE (KEY SUPPLIER PRIVATE)) (KEY CONTRACTOR PUBLIC)))
  (!ASYMMETRIC-DECRYPT-SIGNED SUPPLIER CONTRACTOR
   (SIGNED INVOICE (KEY SUPPLIER PRIVATE)))
  (!ASYMMETRIC-VERIFY CONTRACTOR SUPPLIER
   (SIGNED INVOICE (KEY SUPPLIER PRIVATE)) (KEY SUPPLIER PUBLIC))
  (!NA) (!DONE)))
```

... which involves asymmetric signing, encryption, emailing, decryption and verification. 



### Case 5 – Keys already shared.

Let us finally assume that both authentication and encryption are needed, but this time, we will say that symmetric and asymmetric keys are already shared between the supplier and contractor. The domain and vulnerability assumptions will be given as follows:

```lisp
(defproblem problem1 sec
		(
		; +++++++++++++++++++++++++++++++++++++++++
		; + D O M A I N    A S S U M P T I O N S  +
		; +++++++++++++++++++++++++++++++++++++++++
		
		(has supplier invoice local-drive digital-file)
		
		(has contractor (key supplier contractor shared) local-drive digital-file)
		(has supplier (key contractor supplier shared) local-drive digital-file)
		(has supplier (key supplier private) digital-file)
		(has contractor (key supplier public) digital-file)

		(allow email)
		(allow phonecall)

		(has contractor supplier email-address)
		(has supplier contractor email-address)
		(has contractor supplier phone-number)
		(has supplier contractor phone-number)

   
		; +++++++++++++++++++++++++++++++++
		; +   V U L N E R A B I L I T Y   +     
		; +     A S S U M P T I O N S     +
		; +++++++++++++++++++++++++++++++++

		(is-compromised network)
		(is-compromised contractor mailbox)


	) 

	
	(:ordered
		(sendInvoice supplier contractor)
	)
	
	
) ; end of problem specification
```

Then we will get the following plan:

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1   105.0   105.0      16734     404292     3.125      4.906

Plans:
(((!NA) (!NA) (!NA) (!NA)
  (!ASYMMETRIC-SIGN SUPPLIER INVOICE (KEY SUPPLIER PRIVATE))
  (!SYMMETRIC-ENCRYPT-SIGNED SUPPLIER (SIGNED INVOICE (KEY SUPPLIER PRIVATE))
   (KEY SUPPLIER CONTRACTOR SHARED))
  (!EMAIL SUPPLIER CONTRACTOR
   (ENCRYPTED (SIGNED INVOICE (KEY SUPPLIER PRIVATE))
    (KEY SUPPLIER CONTRACTOR SHARED)))
  (!SYMMETRIC-DECRYPT-SIGNED CONTRACTOR SUPPLIER INVOICE)
  (!ASYMMETRIC-VERIFY CONTRACTOR SUPPLIER
   (SIGNED INVOICE (KEY SUPPLIER PRIVATE)) (KEY SUPPLIER PUBLIC))
  (!NA) (!DONE)))
```

The above plan is similar to case 4 except that the supplier and contractor do not need to install any encryption software or generate keys. Supplier will simply sign and encrypt the invoice, while the contractor will perform decryption and verification.

We can alternatively assume that supplier and contractor might have encryption software already installed, but they have not generated or exchanged keys. In this case, the vulnerability assumptions and security requirements will be similar, however the domain assumption will be given as follows.

```lisp
		; +++++++++++++++++++++++++++++++++++++++++
		; + D O M A I N    A S S U M P T I O N S  +
		; +++++++++++++++++++++++++++++++++++++++++
	  
	  (has supplier invoice local-drive digital-file)
		
		(has supplier encryption-software)
		(has contractor encryption-software)

		(allow email)
		(allow phonecall)

		(has contractor supplier email-address)
		(has supplier contractor email-address)
		(has contractor supplier phone-number)
		(has supplier contractor phone-number)
```

Let us also demand that at no point is there a shared key generated, like we did in case 4.

```text
		; ++++++++++++++++++++++++++++++++++++++++++++++
		; + S E C U R I T Y    R E Q U I R E M E N T S +
		; ++++++++++++++++++++++++++++++++++++++++++++++
		
		(not (ConBreached supplier contractor invoice))
		(and	(AuthSucc supplier contractor invoice)
				(not (AuthBreached supplier contractor invoice))
		)
    ; >>>
    (not (has supplier (key supplier contractor shared) local-drive 
    digital-file))
```


So now, we demand that at no point is there a shared key generated. The planner will respond as follows:

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1   748.0   748.0      38170     920827     8.500     10.752

Plans:
(((!NA) (!NA) (!GENERATE-ASYMMETRIC-KEYS SUPPLIER) (!NA)
  (!EMAIL SUPPLIER CONTRACTOR (KEY SUPPLIER PUBLIC)) (!NA) (!NA)
  (!GENERATE-SYMMETRIC-KEY SUPPLIER CONTRACTOR) (!NA)
  (!BY-PHONECALL-EXCHANGE SUPPLIER CONTRACTOR (KEY SUPPLIER CONTRACTOR SHARED))
  (!TYPE-UP CONTRACTOR (KEY SUPPLIER CONTRACTOR SHARED)) (!NA)
  (!ASYMMETRIC-SIGN SUPPLIER INVOICE (KEY SUPPLIER PRIVATE))
  (!SYMMETRIC-ENCRYPT-SIGNED SUPPLIER (SIGNED INVOICE (KEY SUPPLIER PRIVATE))
   (KEY SUPPLIER CONTRACTOR SHARED))
  (!EMAIL SUPPLIER CONTRACTOR
   (ENCRYPTED (SIGNED INVOICE (KEY SUPPLIER PRIVATE))
    (KEY SUPPLIER CONTRACTOR SHARED)))
  (!SYMMETRIC-DECRYPT-SIGNED CONTRACTOR SUPPLIER INVOICE)
  (!ASYMMETRIC-VERIFY CONTRACTOR SUPPLIER
   (SIGNED INVOICE (KEY SUPPLIER PRIVATE)) (KEY SUPPLIER PUBLIC))
  (!NA) (!DONE)))
```

... which, again, involves asymmetric signing, encryption, emailing, decryption and verification -- but omits installation of software. 





### Case 6 – Information sharing in a highly vulnerable and insecure environment.

Let us, finally, assume that we are interested in confidentiality and integrity of the invoice but the supplier and contractor are operating in a highly vulnerable environment where the network, email, phone, etc. are all compromised. The supplier has an invoice in a printed form, and the domain and vulnerability assumptions are given as follows:

```lisp
(defproblem problem1 sec
		(
		; +++++++++++++++++++++++++++++++++++++++++
		; + D O M A I N    A S S U M P T I O N S  +
		; +++++++++++++++++++++++++++++++++++++++++

		(has supplier invoice physical printed)

		(allow email)
		(allow in-person)
		(allow phonecall)

		(has contractor supplier email-address)
		(has supplier contractor email-address)
		(has contractor supplier phone-number)
		(has supplier contractor phone-number)
		(can-meet contractor supplier)
		(can-meet supplier contractor)
   
		; +++++++++++++++++++++++++++++++++
		; +   V U L N E R A B I L I T Y   +     
		; +     A S S U M P T I O N S     +
		; +++++++++++++++++++++++++++++++++

		(is-compromised network)
		(is-compromised contractor mailbox)
		(is-compromised contractor phone)
		(is-compromised contractor shared-folder)

		(is-bugged supplier)
		(is-bugged contractor)

	) 

	
	(:ordered
		(sendInvoice supplier contractor)
	)
	
	
) ; end of problem specification
```

The above mean that the contractor's mailbox, shared folder (cloud service), and phone are assumed to be compromised (but not their terminal). The supplier needs to find a way to communicate a message to them. Exchange of a shared key is impossible as all the available channels are compromised.

Then we will get the following plan:

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1  -157.0  -157.0       8636     229158     2.078      2.565

Plans:
(((!NA) (!NA) (!NA) (!SCAN SUPPLIER INVOICE) (!NA) (!NA)
  (!IN-PERSON-EXCHANGE-DIGITAL SUPPLIER CONTRACTOR INVOICE) (!NA) (!NA) (!NA)
  (!DONE)))
```

The above plan suggests that the invoice should be scanned and then exchanged in-person in a digital format. Note that in this particular plan many steps are implied for efficiency: once the document is scanned, it is copied to a portable USB medium wherewith the exchange is taking place. The reverse copying happens at the recipients side. 

When we assume however that the physical space of the recipient is compromised, hence the containing USB medium can be extracted. (Notice how we assume the least secure user behavior, with the latter not properly disposing the file and/or medium after use). Thus if we simply add:

```text
  (is-compromised contractor physical)
```

the result will revert to scanning of the document followed by asymmetric encryption.

```text
Defining problem PROBLEM1 ...
---------------------------------------------------------------------------
Problem #<SHOP3::PROBLEM PROBLEM1> with :WHICH = :FIRST, :VERBOSE = :PLANS, OPTIMIZE-COST = T

Totals: Plans Mincost Maxcost Expansions Inferences  CPU time  Real time
           1   825.0   825.0     689351   16990385   164.188    204.900

Plans:
(((!NA) (!NA) (!NA) (!GENERATE-ASYMMETRIC-KEYS CONTRACTOR) (!NA)
  (!EMAIL CONTRACTOR SUPPLIER (KEY CONTRACTOR PUBLIC)) (!NA) (!NA)
  (!SCAN SUPPLIER INVOICE) (!NA)
  (!ASYMMETRIC-ENCRYPT SUPPLIER CONTRACTOR INVOICE (KEY CONTRACTOR PUBLIC))
  (!EMAIL SUPPLIER CONTRACTOR (ENCRYPTED INVOICE (KEY CONTRACTOR PUBLIC)))
  (!ASYMMETRIC-DECRYPT SUPPLIER CONTRACTOR
   (ENCRYPTED INVOICE (KEY CONTRACTOR PUBLIC)))
  (!NA) (!NA) (!DONE)))
```


# Installation Instructions (Windows)

* Install a Common Lisp in your system. We have tried it with [Steel Bank Common Lisp](http://www.sbcl.org/). The [SHOP3 pages](https://github.com/shop-planner/shop3) offer additional recommendations.
* Install SHOP3 following directions [in their github page](https://github.com/shop-planner/shop3). We used the quicklisp installation option solution on windows successfully.
* Save [the listing](https://github.com/genericaylo/submission/blob/main/src/Example.lisp) in a .lisp file, such as ``` Example.lisp```
* Once in sbcl and SHOP3 is loaded (e.g. through ```(load "~/init.lisp")```, run ```(load "Example.lisp")```
* You can make changes to the .lisp file and reload as above; definitions will be replaced.
* Code and framework are currently proprietary and to be used exclusively for reviewing and comprehending associated theoretical papers. Not to be re-distributed or used as a basis for extensions. Please contact the author if questions or requests.
