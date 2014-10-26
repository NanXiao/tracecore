---
layout: post
title: GSM MAP Forward SMS Signaling Introduction
---

When you send a SMS to other people, what happens? The simplified procedure is like this: Firstly, the SMS will arrive at the SMSC (Short Message Service Center), and this is called MO (Mobile Originated) Forward SMS. Then the SMSC will forward the SMS to the recipient user, and this is called MT (Mobile Terminated) Forward SMS.

In GSM [MAP](http://en.wikipedia.org/wiki/Mobile_Application_Part) v1 and V2, there is one signaling used for both MO and MT Forward SMS services: MAP-FORWARD-SHORT-MESSAGE (forwardSM for short, and the opCode is 46). Since MAP v3, There are separated signalings for for MO and MT Forward SMS service: MAP-MO-FORWARD-SHORT-MESSAGE is for MO (MO-forwardSM for short, and the opCode is still 46) and MAP-MT-FORWARD-SHORT-MESSAGE(MT-forwardSM for short, and the opCode is 44)  is for MT.

The famous network analyze tool wireshark is once reported the [bug](https://bugs.wireshark.org/bugzilla/show_bug.cgi?id=1498) about dissecting Forward SMS signalings, and the bug will cause the wireshark "incorrect interpretation of the MAPv2 MT-ForwardShortMessage as a MAPv3 MO-ForwardShortMessage". Form the newest wireahark 1.10.5 source code, we can see the wireshark has done the disposition of differentinating MO and MT scenarios:
  
    static int dissect_invokeData(proto_tree *tree, tvbuff_t *tvb, int offset, asn1_ctx_t *actx) {
        ......
      case 44: /*mt-forwardSM(v3) or ForwardSM(v1/v2)*/
        if (application_context_version == 3)
          offset=dissect_gsm_map_sm_MT_ForwardSM_Arg(FALSE, tvb, offset, actx, tree, -1);
        else {
          offset=dissect_gsm_old_ForwardSM_Arg(FALSE, tvb, offset, actx, tree, -1);
        }
        break;
      case 46: /*mo-forwardSM(v3) or ForwardSM(v1/v2)*/
        if (application_context_version == 3)
          offset=dissect_gsm_map_sm_MO_ForwardSM_Arg(FALSE, tvb, offset, actx, tree, -1);
        else {
          offset=dissect_gsm_old_ForwardSM_Arg(FALSE, tvb, offset, actx, tree, -1);
        }
        break;
    
        ......
    }


The primary parameters in Forward SMS signalling:
SM RP DA: In MO service, it is the SMSC address. In MT service, it is the IMSI of recipient user in most cases.
SM RP OA: In MO service, it is sender phone MSISDN. In MT service, it is the SMSC address.
SM RP UI: The short message transfer protocol data unit, and it contains the actual SMS content and other information.
More Messages To Send: This parameter is used since MAP v2, and only in MT service. This indicates whether the SMSC has more messages to send to recipient phone.