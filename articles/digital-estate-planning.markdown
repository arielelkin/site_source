---
layout: page
title: "Digital Estate Planning"
date: 2017-02-24 00:00
comments: false
sharing: false
footer: false
---


# Motivation

Our wishes and rights push us to build tools that preserve our privacy during our life span, but these very tools may hinder our wishes and rights when we are no longer here to control them.

Traditional technologies protect a user’s privacy by upholding the basic information security directive that the user, and only the user, have sole exclusive custody and knowledge of his keys. They also assume, both in theory and in practice, that the user is immortal and perennially healthy. The corollary of many of these policies is that a (deceased) user is buried with his keys.

This is an awkward situation for many people: those of us who are both privacy-conscious and uncomfortable at the prospect of their keys following them to the grave. (Note that death is not the only life event that gives rise to this problem: events such as physical incapacitation or incarceration are equally problematic). Yet no widely-used technology offers any provisions for passing on the keys to the entirety of a person’s digital assets after their death. Most workarounds, such as sharing a copy of one’s private key with someone else, entail security risks and privacy curtailments.

Indeed, a digital asset can be said to be part of one’s digital estate if one has private have access to it. But passing on one’s digital estate after one’s death is an issue that has received little attention from academic Computer Science, and little originality from the private sector. Prior research in the field of Computer Science has focused on best practices in passing digital assets priorly known to or under custody of a third-party, viz. one’s pre-existing social media account; but so far, no satisfactory overarching strategies have been proposed that can handle the entirety of a person’s digital assets.

The aim of the present article is to suggest a formal framework for thinking about digital estate planning: A **digital estate** is a set of digital _assets_ with pre-assigned _persistence types_. Its **planning** refers to the users and a predesignated group of _executors_ following a predefined _protocol_ for the processing the assets after the user's death.

We will discuss acceptable criteria for reliable, user-friendly, and privacy-enhancing digital estate planning protocols and tools, and argue that these criteria can be satisfied by threshold cryptosystems.

# Assets

A person’s digital estate is the sum of the digital assets that he has private access to. It includes files stored remotely or locally on his devices, as well as the entitlements granted by the person’s account credentials, passwords, or keys.

Digital files are a type of digital asset, and can be of two kinds as far as a digital estate is concerned. Those not available to the public to which the person has read or write access (such as the person’s personal documents and images), and those available to the public to which the person has private write access (such as images published on the person’s blog).

Assets also include account credentials and the corresponding assets, persona, and entitlements they grant access to. For example, a person’s YouTube account credentials grant access to
the videos the person uploaded to YouTube, the capability to remove previously uploaded videos or upload new ones, as well as the online identity used by the person to carry out various tasks within YouTube, such as posting comments on videos. Another typical example is a person’s email account credentials, which grant access to the person’s email correspondence, as well as the capability to permanently delete email messages or send new ones.

Assets can take many other forms. They include the person’s private encryption keys (e.g. those used in PGP or SSH), the password for logging into the person’s computer, a mobile device PIN code, or web domain names.

# Persistence Types

For every asset or group of assets in his digital estate, a person, i.e. the _testator_, must specify if and how the asset should outlast him.
Assets may be assigned one of the following persistence types:
1. _No Persistence_: the asset must be permanently destroyed as soon as possible after the person’s death.
2. _Private Immutable Persistence_: the asset must be preserved in the state it was left by the person, its access must be restricted to a predesignated group of people.
3. _Public Immutable Persistence_: the asset should be made available to the public and must be preserved in the state it was left by the person.
4. _Public Mutable Persistence_: the asset should be made available to the public and may be modified by the public.

(A testator's wishes with regards to a particular asset’s persistence may not necessarily fit with one of the above types, in which case he should simply specify the way he wishes the asset to persist; indeed the purpose of the proposed persistence types is to have the user clarify his intent with regards to the asset’s persistence.)

# Executors

For every asset or group of assets in his digital estate, a testator must designate one or more persons or organisations to whom ownership will be transferred for the purposes of assigning it the required persistence type, and, if applicable, managing it thereafter.

# Digital Testament

A _digital testament_ is a text document that:

* Rounds up the testator's estate
* Specificies individual digital assets' persistence types
* Designates executors

# Requirements Of Privacy-preserving Digital Estate Planning Strategies And Protocols

A digital estate planning strategy or protocol is _comprehensive_, _privacy-preserving_, _reliable_, and _convenient_ if it satisfies the following criteria:

## Comprehensiveness

It clearly specifies the entirety of the estate assets (to the testator's satisfaction), their desired persistence types, and designates a set of executors.

## Privacy-Preservation
1. it demonstrably does not grant access to or knowledge of any part of the estate to any entity other than the testator before his death.
1. it demonstrably grants access to the estate’s assets to just the executors predesignated in the estate, but to no executor individually.
1. it can be authenticated as coming from the stated testator.

## Convenience
The testator can, at any point and without compromising ease of use or privacy:

1. add, edit, or remove assets
1. edit their persistence types
1. edit the executors list.


# Protocols

## Centralised Fiduciaries

Many online services, such as Securesafe, are designed to assist with digital estate planning. The testator typically stores account credentials and files on the service’s cloud and designates beneficiaries who will be given access to the assets in his account in the event of his death.

These services usually satisfy **Convenience** and **Comprehensiveness** criteria, but fail to satisfy the **Privacy-Preservation** criterion: the testator must share knowledge of and potentially access to his digital estate with the service. And the majority being closed-source, they require an undue amount of trust placed on a third-party.


## Secret Sharing Schemes

Secret sharing schemes, in which several parties must cooperate in the decryption process, without the need of a trusted central entity, are adequate for the purposes of privacy-preserving digital estate planning.

In a basic implementation, the testator is responsible for dividing the testament into individual encrypted shares and distributing them among his executors.

Distributing the shares to several parties increases the scheme’s security, as no individual executor can access the cleartext testament without colluding with the other executors. An attacker has to compromise either the testator's or all executors' shares in order to gain access to the testament.

This also performs better than “dead man’s switch” time-based schemes in granting access to the document at the person’s death, for if the shares are distributed among executors relatively close to the person, the executors can personally satisfy themselves that the person is indeed deceased and collaborate to decrypt the testament.

An additional advantage of such a scheme is that can readily be implemented so as to offer information-theoretic security via the use of Shamir’s Secret Sharing, an encryption algorithm designed for such use cases. (This implies that an attacker would not be able to break the encryption in theory and in practice, regardless of the amount of computing power used, as long as he does not compromise a sufficient number of shares).

Unlike Centralised Fiduciary systems, such a Secret Sharing Scheme fully satisfies the **Privacy-Preservation** and **Comprehensiveness** criteria above.

However, it falls short of satisfying the Convenience criteria, for our digital estates evolve quickly: we regularly sign up for new services, accumulate new assets, change our passwords, etc. This scheme would require the testator to re-encrypt and re-distribute his testament every time he wishes to edit it; this compromises ease of use.

### Suggested Implementation of Secret Sharing Schemes for Digital Estate Planning

A serial combination of symmetric-key cryptography and secret sharing schemes can improve the usability of secret sharing schemes for digital estate planning. The following sequence is suggested:

1. Alice encrypts her digital testament using a symmetric encryption algorithm, splitting the key according to a threshold cryptosystem
1. Alice publishes the encrypted testament.
1. To each executor, Alice gives one of the shares of the encrypted key.

Under this scheme, Alice may update her testament at will without having to notify her executors, and the **Convenience** criteria is fully satisfied. Alice must now trust a third-party for the broadcasting of her testament, but nowadays there exists many adequate decentralised file storage systems (such as IPFS, Storj, or Tahoe-LAFS).

# Changelog

You can track this article's edits [here](https://github.com/arielelkin/site_source/commits/master/articles/digital-estate-planning.markdown).

# Acknowledgements

My thanks to Alejandro A.F. Japkin (NINA Science co-founder), G. Moore, J.M. Lindemann., and Pablo Vasquez.

# References

* Micklitz, S., Ortlieb M., and Staddon, J. 2013. “I hereby leave my email to…”: Data Usage Control and the Digital Estate. In _Security and Privacy Workshops (SPW)_, IEEE. <http://dx.doi.org/10.1109/SPW.2013.28>

* Tsaasan, A. M., Kusumakaulika, N., and Brubaker, J. R. 2015. Design Considerations and Implications in Post-Mortem Data Management. In _iConference 2015 Proceedings_. <http://hdl.handle.net/2142/73729>

* Brubaker, J. R., Dombrowski, L. S., Gilbert, A. M., Kusumakaulika, N., & Hayes, G. R. 2014, April. Stewarding a legacy: responsibilities and relationships in the management of post-mortem data. In _Proceedings of the SIGCHI Conference on Human Factors_ in Computing Systems (pp. 4157-4166). ACM. <http://dx.doi.org/10.1145/2556288.2557059>

* Bahri, L., Carminati, B., & Ferrari, E. 2015, August. What happens to my online social estate when I am gone? An integrated approach to posthumous online data management. In _Information Reuse and Integration (IRI)_, 2015 IEEE International Conference on (pp. 31-38). IEEE. <http://dx.doi.org/10.1109/IRI.2015.16>

* Shamir, A. 1979. How to share a secret. _Communications of the ACM_, 22(11), 612-613. DOI=http://dx.doi.org/10.1145/359168.359176
Pedersen, T. P. 1991, April. A threshold cryptosystem without a trusted party. In Advances in Cryptology—EUROCRYPT’91 (pp. 522-526). <http://dx.doi.org/10.1007/3-540-46416-6_47>

* Digital Estate Planning Online Services List. Retrieved April 2016. The Digital Beyond. <http://www.thedigitalbeyond.com/online-services-list/>

* Rijmenants, D. 2009. Secret Splitting: A Practical and Secure Way to Delegate Keys. <http://users.telenet.be/d.rijmenants/papers/secret_splitting.pdf>

* Arnold, C. 2013. How to Manage Your Digital Afterlife. In Scientific American. <http://www.scientificamerican.com/article/how-to-manage-your-digital-afterlife/>

* Walter et al. 2011. Does The Internet Change How We Die And Mourn? An Overview. <https://www.scribd.com/document/119759720/Does-the-Internet-change-the-way-we-die-and-mourn-An-Overview>

* Walker, R. 2011. Cyberspace When You’re Dead. <http://www.nytimes.com/2011/01/09/magazine/09Immortality-t.html>

* Doctorow, C. 2009. When I'm dead, how will my loved ones break my password? <http://www.theguardian.com/technology/2009/jun/30/data-protection-internet>
