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

This is an awkward situation for many people: those of us who are both privacy-conscious and uncomfortable at the prospect of their keys following them to the grave. (Note that death is not the only life event that gives rise to this problem: events such as permanent incapacitation or incarceration are equally problematic). Yet no widely-used technology offers any provisions for passing on the keys to the entirety of a person’s digital assets after their death. Most workarounds, such as sharing a copy of one’s private key with someone else, entail security risks and privacy curtailments.

Indeed, a digital asset can be said to be part of one’s digital estate if one has private have access to it. But passing on one’s digital estate after one’s death is an issue that has received little attention from academic Computer Science, and little originality from the private sector. Prior research in the field of Computer Science has focused on best practices in passing digital assets priorly known to or under custody of a third-party, viz. one’s pre-existing social media account; but so far, no satisfactory overarching strategies have been proposed that can handle the entirety of a person’s digital assets.

The aim here is to suggest a new formal framework for thinking about digital estate planning: A digital estate is a set of _assets_ with pre-assigned _persistence types_, a predesignated group of _executors_, and a _protocol_ for the user and the executors to follow.

We will discuss acceptable criteria for reliable, user-friendly, and privacy-enhancing digital estate planning protocols and tools, and argue that these criteria can be satisfied by threshold cryptosystems.

# Assets

A person’s digital estate is the sum of the digital assets that a person has private access to. It includes files stored remotely or locally on the person’s devices, as well as the entitlements granted by the person’s account credentials, passwords, or keys.

Digital files are a type of digital asset, and can be of two kinds as far as a digital estate is concerned. Those not available to the public to which the person has read or write access (such as the person’s personal documents and images), and those available to the public to which the person has private write access (such as images published on the person’s blog).

Assets also include account credentials and the corresponding assets, persona, and entitlements they grant access to. For example, a person’s YouTube account credentials grant access to
the videos the person uploaded to YouTube, the capability to remove previously uploaded videos or upload new ones, as well as the online identity used by the person to carry out various tasks within YouTube, such as posting comments on videos. Another typical example is a person’s email account credentials, which grant access to the person’s email correspondence, as well as the capability to permanently delete email messages or send new ones.

Assets can take many other forms. They include the person’s private encryption keys (e.g. those used in PGP or SSH), the password for logging into the person’s computer, a mobile device PIN code, or web domain names.

# Persistence Types

For every asset or group of assets in his digital estate, a person must specify if and how the asset should outlast him.
Assets may be assigned one of the following persistence types:
1. _No Persistence_: the asset must be permanently destroyed as soon as possible after the person’s death.
2. _Private Immutable Persistence_: the asset must be preserved in the state it was left by the person, its access must be restricted to a predesignated group of people.
3. _Public Immutable Persistence_: the asset should be made available to the public and must be preserved in the state it was left by the person.
4. _Public Mutable Persistence_: the asset should be made available to the public and may be modified by the public.

(A user’s wishes with regards to a particular asset’s persistence may not necessarily fit with one of the above types, in which case he should simply specify the way he wishes the asset to persist; indeed the purpose of the proposed persistence types is to have the user clarify his intent with regards to the asset’s persistence.)

# Executors

For every asset or group of assets in his digital estate, a person must designate one or more persons or organisations to whom ownership will be transferred for the purposes of assigning it the required persistence type, and, if applicable, managing it thereafter.

# Requirements Of Privacy-preserving Digital Estate Planning Strategies And Tools

A digital estate planning strategy is comprehensive, privacy-preserving, reliable, and convenient if it satisfies the following criteria:

## Comprehensiveness

It clearly specifies the entirety of the estate assets (to the person’s satisfaction), their desired persistence types, and designates a set of executors.

## Privacy-Preserving
1. it demonstrably does not grant access to or knowledge of any part of the estate to any entity other than the person before his death.
1. it demonstrably grants access to the estate’s assets to just the executors predesignated in the estate, but to no executor individually.
1. it can be authenticated as coming from the stated person.

## Convenience
The person can, at any point and without compromising ease of use or privacy:

1. add, edit, or remove assets
1. edit their persistence types
1. edit the executors list.


# Implementations

## Centralised Fiduciaries

Many online services, such as Securesafe, are designed to assist with digital estate planning. The user typically stores account credentials and files on the service’s cloud and designates beneficiaries who will be given access to the assets in his account in the event of his death.

These services usually satisfy criteria B and E, but fail to satisfy the other criteria. Crucially, they perform poorly from a privacy standpoint: the person must share knowledge of and potentially access to his digital estate with the service. And the majority being closed-source, they require an undue amount of trust placed on a third-party.

## The Case for Threshold Cryptosystems
It will be argued that threshold cryptosystems, in which several parties must cooperate in the decryption process, without the need of a trusted central entity, are adequate for the purposes of privacy-preserving digital estate planning. Shamir’s Secret Sharing is an example of an algorithm in cryptography designed for this kind of use case, and offers information-theoretic security.

Under such a scheme, a user would write a document rounding up his estate, specify individual asset persistence types, and list executors. This document would then be encrypted using one or more keys to be shared among the executors.

Distributing the keys to several parties increases the scheme’s security, as no individual executor can access the plaintext document without collaborating with the other executors, and an attacker would have to compromise either the person’s or all executors’ keys in order to gain access to the document. It also performs better than “dead man’s switch” time-based schemes in granting access to the document at the person’s death, for if the keys are shared among executors relatively close to the person, the executors can personally satisfy themselves that the person is indeed deceased and collaborate to decrypt the document.
