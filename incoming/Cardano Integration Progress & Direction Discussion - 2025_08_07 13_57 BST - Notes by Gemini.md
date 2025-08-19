Aug 7, 2025

## Cardano Integration Progress & Direction Discussion \- Transcript

### 00:00:00

   
**Michał Skowron:** Hello.  
**Mike Clay:** So, uh, this is the doc as it's done so far, the integration summary. Um, so I think what I'd like to do is um whether or not we keep these phases.  
**Michał Skowron:** Mhm.  
**Mike Clay:** Yeah, these phases are just kind of an arbitrary sort of thing. So I suspect Yeah, we might want to decide what's a priority, but what I'd like to do is like just focus on what Yeah.  
**Michał Skowron:** to be to be honest I'd rather have a plan of tasks as an output and then because if you have plan of task then you can use AI again to just make them more when you're working on it you can use it uh you know you know to make it more detailed I guess.  
**Mike Clay:** Yeah. Yes. You need to know what features you want though first, right? So, um, so looking at these, I mean, this is just a start point. So for the second phase just just run down this. So we need an RPC implementation which we don't have at the moment.  
   
 

### 00:03:16

   
**Michał Skowron:** Yeah, that's that that's obvious. Wait.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So, what is is that first one?  
**Mike Clay:** So what what what I propose is regardless of whether these are off the mark or not, let's go down the list, decide whether we want to keep these and let's have a discussion about what each one looks like. We want to keep it.  
**Michał Skowron:** Why did you skip it?  
**Mike Clay:** This is what's been done already. So the whether we I'm not I'm not so interested in how how we number the phases.  
**Michał Skowron:** Okay.  
**Mike Clay:** I'm just interested in the tasks that have been identified in this document. Like I'd like to qualify a whether they are things that we need to do and b if they are then let's talk about what that looks like. So um so the RPC client um what what do we use for that in the other swap repo?  
**Michał Skowron:** So there already is an obvious client at least in the current in the version of partner chains that we use. I'm not sure if they have removed it later but even if they did we can use it.  
   
 

### 00:04:17

   
**Mike Clay:** Yeah.  
**Michał Skowron:** Now I don't see a reason to not not to. And this y def kit I'm using for local cardano already has obvious option.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So it's uh I'm not sure if it's going to be you know a a production solution. It might be but I don't think we should care. We we go we want to have a quick uh quick card implementation.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So yeah.  
**Mike Clay:** Yeah. We're not really targeting production right now, so  
**Michał Skowron:** Yeah. So and it might actually it might be production good for production. So, I think it's it's a no-brainer here. Just uh implement those RPC clients that we have or maybe modify them if if needed for the workflow. But we definitely need RPC implementation. Well, do we need to base ourselves on some use cases here?  
**Mike Clay:** Um, yeah. Well, I mean, we needed witnessing for Ethan BTC though, right?  
**Michał Skowron:** Because my my first use case is this rewards. is the simplest one and it doesn't for example it doesn't need witnessing and the other use case might be just a swap yeah because we need this we don't need with nursing but I mean okay we kind of need witnessing probably for I mean with nursing is going to be needed for for a lot of stuff but and But the basic flow can be done without witnessing.  
   
 

### 00:05:54

   
**Michał Skowron:** Just give him an address make and just send transaction and check on chain.  
**Mike Clay:** Yeah. Yeah.  
**Michał Skowron:** That's and that's why I like it because I can focus on this use case and then you can just support witnessing which is in a separate place in the code.  
**Mike Clay:** Yeah. Okay. So I mean as far as this this kind of dock goes witness implementation is identified a bit further down. Um so we can we can forget about that for now anyway. Witness implementation.  
**Michał Skowron:** Okay, let's let's go and about this RPC client.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So for that use case with transactions, I mean with rewards without witnessing, I just need to to implement submit transaction and those other functions will be needed for witnessing.  
**Mike Clay:** Yeah. Yeah. Okay. Um I mean I'm sure the tool that you're using once you specify that it's using an OMIOS and uh give it an idea about the actual should be able to infer what's needed I would say.  
**Michał Skowron:** Yeah, it's going to be super simple like Yeah, I suppose that Yeah, exactly.  
   
 

### 00:07:03

   
**Michał Skowron:** It should be like those implementations if we have this client obviously and running. I think it's a really good also it's should be a really simple task for AI.  
**Mike Clay:** Yeah.  
**Michał Skowron:** I mean it's one of those task where it should excel if it has good so and another so so so we definitely need it address system I I'm not sure in which exact parts of the app but uh  
**Mike Clay:** Yeah. Yeah.  
**Michał Skowron:** we probably need some uh address derivation logic like for deposit channel stuff like that might be even needed for that basic no maybe not I'm not sure exactly but it might also be needed for that uh reward um  
**Mike Clay:** Yeah.  
**Michał Skowron:** uh for other rewards to to work.  
**Mike Clay:** Yeah. Okay. I'll um what I'll do is I'd like to clarify the address system item then and and find out what the what the impact of that is and uh what further derivations needed. Bear in mind like I'm recording this and transcripting it. So I can get out directly just by mentioning it.  
   
 

### 00:08:03

   
**Michał Skowron:** Yeah, I I I know. Yeah, I know. I know. I mean it's visible also in the German and also we we talked about it before so I know uh anyway address system is definitely yeah now I don't think it is going to be needed if you want to  
**Mike Clay:** Yeah.  
**Michał Skowron:** have deposit channels like for that first implementation I'm not sure yet so actually this is kind of to figure out how to do Yeah, this is what I'm working on right now.  
**Mike Clay:** Yeah. Yeah. Cool. Okay. Uh transaction construction, CBO transaction building. Okay. So, you're you're working on that right now. Okay.  
**Michał Skowron:** Yes.  
**Mike Clay:** Uh fee calculation.  
**Michał Skowron:** So, we probably we didn't we need it like looking at how is it done in other chains.  
**Mike Clay:** Uh  
**Michał Skowron:** there is witnessing which uh witnesses that the fee and it's taken from from those witness values on chain. So this is going to be part of witnessing.  
**Mike Clay:** right? So, so fee calculation and witnessing are intimately linked, which kind of if we're not doing witness implementation until later, we won't be able to do fee calculation until that's done.  
   
 

### 00:09:12

   
**Michał Skowron:** Yes. I mean we can fix you know for the test when then you can we can fix it and it's going to work.  
**Mike Clay:** Yeah.  
**Michał Skowron:** But it's part of witnessing UTX management it's also needed.  
**Mike Clay:** Yeah. Yeah.  
**Michał Skowron:** So we will probably need the same I would just not think and use do it in the same way as it's in Bitcoin. So they actually rotate from the old keys through UTXO consolidation.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So we will have to do that as well. You see the XO consolidation or Cardano there what for the for the no it's not like that you like I don't know what do you mean so basically what UTXO consolidation is that  
**Mike Clay:** Okay. Can we um can we do that through the Omnio client?  
**Michał Skowron:** for for an address when people make transactions there are a lot of UTXOs created and if the number of XO is week then all the operations you know are fragmented takes take more time and also UTXO are signed by some keys and if you so to change the ownership you need to create another transaction so if we do and we want so the way it's done currently  
   
 

### 00:10:29

   
**Mike Clay:** Yeah. Yeah. Sorry. What I'm saying is um has Ognio got an API for accessing and managing UTXOs?  
**Michał Skowron:** for let's say Bitcoin in for Bitcoin is that we witness each block and and from each block we're taking all the transactions. We filter them based on what we observe and we record the the UTXOs we own on chain.  
**Mike Clay:** Yeah. Yeah.  
**Michał Skowron:** So we don't need any other sophisticated way to do that.  
**Mike Clay:** Ultimately, ultimately the way we the way we get the witness data is viaos client ultimately because we need to look at the chain or do we using DB sync for that?  
**Michał Skowron:** There might be some more sophisticated ways like using DB sync indexes but Yes. But but no, I mean if so what I'm proposing is to use the same way as for Bitcoin. For Bitcoin, we just want to query the blocks and they the transactions in the blocks.  
**Mike Clay:** Yeah. Yeah.  
**Michał Skowron:** So for that you don't need any sophisticated app but get the block obvious.  
   
 

### 00:11:44

   
**Mike Clay:** using what? Agnio.  
**Michał Skowron:** Yes, it's going to suff suffice for what we need at least in this first unoptimized way, which is fine.  
**Mike Clay:** Yeah, that's what I was looking for. Yeah. Okay. Yeah. So, what I'm saying is the API that it's got will be the lowest level of all of these operations. Yeah. Yeah. Okay. So, where whereabouts would we need to use dbsync for any of these?  
**Michał Skowron:** We won. We done.  
**Mike Clay:** Okay, that's good. That's good to know. Uh because if we've got if we only got one client that we need to use for this, then it kind of yeah simplifies things a little bit. Um DB sync's got its own issues.  
**Michał Skowron:** I mean we we we need to use DB sync for current and partner chains operations but if we by some chance have the separate you know because we were talking about it with Paul we consider to have a se completely separate chain which handles these uh crosschain operations like you know just take take uh let's say change the index or change the coins change at at Cardano and just put it there to to be used.  
   
 

### 00:12:48

   
**Mike Clay:** H. Yeah.  
**Michał Skowron:** It's also a possible solution. So in such case we by not using DB sync we are like you know we it's it would be a big advantage for us not to use PDB sync I mean it's nice I think I have  
**Mike Clay:** Yeah. Yeah. Okay. All right. Let's move on to the next one. Batch operations, I guess. Do we need to do batching? It's It's good for optimization.  
**Michał Skowron:** it in the Cardano app API but we don't have to I wouldn't care in about it in the first face I mean or later like we we will have an operational implementation without that because this is optimization  
**Mike Clay:** Yeah. So batch operations can go can move to the next phase then. Yeah. Yeah. Of course. Yeah. So what about deposit withdrawal?  
**Michał Skowron:** this yeah this uh I'm not sure how the current logic sense so if we just implement those things we were talking about before it might just work.  
   
 

### 00:14:00

   
**Michał Skowron:** I have no idea on what I mean if we have the you know the uh what was those the uh deposit channels vaults working uh the rotate committee rotation working then with nothing then it's  
**Mike Clay:** Yeah. H. Yeah.  
**Michał Skowron:** possible that it's just going to work so I'll say this thing is to be revisited after we finish the previous ones Or we can have two approaches for that.  
**Mike Clay:** Yeah. And finally for this one, error handling. I think that's an optimization really.  
**Michał Skowron:** Either like look at it later again or just work in a or just think about it during implementation. Actually when using AI they a lot of times it creates nicer error handlers than I would create from the start.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So so we might not Um if it will I mean yeah but uh we can leave it at the end to just check the code for those panics this panics and other stuff.  
**Mike Clay:** Yeah. Yeah. We can leave it in this phase then and just get AI to handle the error handling.  
   
 

### 00:15:00

   
**Mike Clay:** Uh we get Yeah. Yeah. Yeah. Okay. So, I'm not going to go through phase three and four just because we've got enough here to give us tasks for the next week or two weeks, I think.  
**Michał Skowron:** Yeah, I also think wait let's let's me think from the start what happens  
**Mike Clay:** So, um is there anything that you think would go in the in what's labeled as phase two here that we've is missing?  
**Michał Skowron:** let's say when we have the use case of uh paying rewards in ADA. First, we need to have a vault. In the vault, we need to have some ADA. So, again, we going to need like to have a full test.  
**Mike Clay:** Cardano.  
**Michał Skowron:** We going to have some kind of script, you know, like in Ethereum probably which will Yeah.  
**Mike Clay:** We need we need the Cardano equivalent of the Ethereum side, the contract script and the vault.  
**Michał Skowron:** So I mean we have those those things will add the vault capability but you know to to put to to schedule rewards first we have to have a there and if you remember it's going to be a similar  
   
 

### 00:16:19

   
**Mike Clay:** Yeah. So, we need a vault capability in in Cardano. Yeah.  
**Michał Skowron:** way as it was done in Bitcoin we don't have a contract so in bitcoin we had to we have a public key of the first committee.  
**Mike Clay:** Yeah.  
**Michał Skowron:** This is this change I added you know with those DB DBs on the last PR.  
**Mike Clay:** Yeah. Yeah.  
**Michał Skowron:** So we we need a similar if we will do it in the same way with deposit channels then it's the same same situation as in Bitcoin if you remember there was a lot of scripting to get the address then uh but we had eventually we had it in the scripts am I right? So probably we might just base the base but it's another task also to have this but it I I mean I mean I'll say it's very useful  
**Mike Clay:** Yeah. Yeah. Yeah. So the Cardano side logic is another task which is not obviously not going to be in this list because it's not part of this kind of code base but I would say yeah of course yeah  
   
 

### 00:17:32

   
**Michał Skowron:** for like putting everything together because if you have the script which which uh you start the the Yasi dev kit you top up the correct amount with rewards then you can demo it simply I have  
**Mike Clay:** so have you used the Yassi devkit successfully.  
**Michał Skowron:** run it it was been creating blocks but I don't have any witnessing created so and also I don't have the RPC client implemented so there wasn't any I didn't have an occasion to check the connectivity from the last  
**Mike Clay:** Yeah.  
**Michał Skowron:** up.  
**Mike Clay:** Yeah. Is it would it be possible for you to get a like a read me and a like a a quick startup kind of Okay.  
**Michał Skowron:** It's just use me. It's I didn't use anything other than the so it's I just followed it and it was quite straightforward and and I wouldn't do read mem yet because I don't know how will it work you know uh with yeah  
**Mike Clay:** Okay. Fair enough. Yeah. Yeah. Yeah. That's fine. As long as it's straightforward.  
   
 

### 00:18:34

   
**Mike Clay:** If I' if I try and set it up and I've got any questions, I'll I'll ask you.  
**Michał Skowron:** it was like quick I don't I don't think if I like I didn't I don't think if I spent more than 15 minutes on like checking the oh how to start that way something like Wait because we started  
**Mike Clay:** Yeah. Yeah. Okay. Uh Okay. So that that's the Cardano side uh logic  
**Michał Skowron:** thinking about what is needed. So we started from the deposit. Then if we have this deposit and we have and we have correct crypto which kind of already goes through the code now fails on the sending transaction. So we will have to have the RPC client end which we have here. uh to send transaction and it should be actually and then with nothing to progress f far further to see that it was sent like you know later but the first part and descending rewards will actually work with this and those scripts helping scripts.  
**Mike Clay:** Yeah.  
**Michał Skowron:** If you want to go farther like multiple to go for correctly for multiple uh rotations then we need to witness that the rotations happen and things like that.  
   
 

### 00:19:46

   
**Michał Skowron:** So we need witnessing for that which we have here although not I think that we are witness should be in a phase two I mean and by this phase three testing is I'll say it's going to be some  
**Mike Clay:** Um I think perhaps then we should bring the witness implementation to phase two because then phase three can just be testing and validation. Yeah.  
**Michał Skowron:** integration testing because like I'll say those unit tests those the tests what I found you know if you can write good tests then the AI works much better.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So every time I would start from testing and I have even like a command for cloud code which remains it to make to do TDD because it forgets about this.  
**Mike Clay:** Yeah. Right. Yeah. So you so you're actually asking the the tool to do TDD. You're asking it to do a TDD style method. I mean, does that suppose that's valid?  
**Michał Skowron:** Yeah.  
**Mike Clay:** Yeah. I don't I mean, I've never I mean, I've never considered whether TDD has benefit for AI as well as humans.  
   
 

### 00:20:43

   
**Mike Clay:** I mean, it does for humans, but that's Yeah.  
**Michał Skowron:** It looks like it's more for AI even like I've been reading a lot of a lot of oh this is my cloud MD for like general code MD it's it's still like a draft but I have like this on top about about TDD and but still I have to remember I have to in when I when I start each session I tell him to read this because otherwise he forgets.  
**Mike Clay:** Um okay. Yeah. How are you finding claw code then?  
**Michał Skowron:** It's nice. Again, I still I still I still need to test more features like especially those commands, those rows, multi- aents. I didn't have time to to to test it yet.  
**Mike Clay:** Yeah.  
**Michał Skowron:** But uh you know unfortunately this coding is not as easy and straightforward as people say to have a complex code you need to put a lot of work but in different places I mean  
**Mike Clay:** Yeah. Yeah. Yeah. the work. There's no less work.  
   
 

### 00:21:57

   
**Mike Clay:** It's just that the abstraction level is different that you need to put the work in. Yeah.  
**Michał Skowron:** I think it's like I mean if you if you have because those planning this stuff you usually don't have it in normal projects or you have to do it at the end at the like the quality of it is worse. So if you start with good quality planning documentation then you get it really you know really nice is nice for the production uh we're going to production.  
**Mike Clay:** Yeah. Cool. Okay. Um, so I think it's sensible to create tasks from these.  
**Michał Skowron:** Yeah and what we said about the scripts also. So we have a few tasks.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So what I think because now like this task I'm taking I'm just dictating it by what errors I have when I run my setup.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So what I see from what we've thought I think you you could be free to take either with nesting or those scripts with def kit those scripts are easier and shorter.  
   
 

### 00:23:02

   
**Mike Clay:** Yeah.  
**Michał Skowron:** So yeah, I only try I'll only do the sense transaction for now.  
**Mike Clay:** Yeah. Are you going to are you going to do the look at the RPC client integration? Okay. Because Yeah.  
**Michał Skowron:** So you you you can you can if you want you can you can fill other methods or you know if you will be doing that you can just fill all of them. Even if we just clash ourselves in this one method, it's not a lot of uh clash.  
**Mike Clay:** Yeah. Let me So, okay. Let me go away and let me create some tasks for these and I'll add the witness implementation. Yeah.  
**Michał Skowron:** Okay. Oh, I I am al also wondering because also there are some some projects which helps you know task which makes which integrate clothes maybe clothes maybe those are MCPS with task management systems. So you know you create tasks and then you just people have it like that that you create task let's say on GitHub and have MCP for cloud and commands where you style cloud take take this Mhm.  
   
 

### 00:24:08

   
**Mike Clay:** Yeah, I think I I did think about that, but I think for now I don't think we've got that level of integration. So, what I'm going to do is I'll get the AI to just create the the text for the title and the description for the tasks for each of these. And uh I'm just going to have to do it manually for now.  
**Michał Skowron:** Okay. And what about the rest of this file? Because this face was actually quite handy. We can add the scripts to this, but the rest we didn't review yet.  
**Mike Clay:** Yeah.  
**Michał Skowron:** I wouldn't kind of maybe add some marks.  
**Mike Clay:** Yeah.  
**Michał Skowron:** So even because I don't want to have f also files which make AI AI job harder.  
**Mike Clay:** I'm gonna I'm split this file because that it talks about what's been created and modified.  
**Michał Skowron:** So yeah.  
**Mike Clay:** I'm going to move this somewhere else, I think. Uh because I'd like to have a file that's just for planning just just for like this planning logic.  
**Michał Skowron:** So I would either not like remove it from the branch for now or make it low less like only planning things and I would say only most like I would keep commit only those parts which we are approved  
   
 

### 00:25:05

   
**Mike Clay:** Yeah.  
**Michał Skowron:** like which we know. So cuz otherwise I also have this like a few files which I think I put a little bit too expensive they are and then they mess up like the more they they mess up with the AI  
**Mike Clay:** Yeah. Yeah. So let's have a look. I think so what what we'll do is we'll I I'll keep the implementation stages and next steps in a in a document all on its own and then the the other things which yeah is useful information I'll I'll  
**Michał Skowron:** I'm Not sure.  
**Mike Clay:** move that Yeah, probably not even to be fair.  
**Michał Skowron:** I mean those types this is like tech this implementation details with code I mean and they are hard to m maintain. You keep that then you change the code then I forgets to modify it. I even have in this to to update update documentation.  
**Mike Clay:** Yeah. Yeah. Okay. Yeah, I'll cut this.  
**Michał Skowron:** But I mean the f the structure of the files is sometimes useful for AI.  
   
 

### 00:26:06

   
**Mike Clay:** Yeah, I'll cut this right down.  
**Michał Skowron:** That's true. But you also need to keep that updated.  
**Mike Clay:** Yeah. So what what I find is that if you don't specify otherwise, the AI can be overly verbose and you don't need it to be. And also it can be can be a problem in the future because it's just bloat.  
**Michał Skowron:** Yeah.  
**Mike Clay:** The bloat can it can rot the context.  
**Michał Skowron:** Yeah, if you get and yeah and if you have bloated like that first you get overhand with reviewing reading that and then and the but the best thing is that AI also gets gets too much in the context  
**Mike Clay:** Yeah.  
**Michał Skowron:** and it's it stops following it it just stops using it and it gets very useless.  
**Mike Clay:** Yeah. Yeah. So, um, so for the benefit of the AI that's going to listen to this conversation, uh, let's, uh, let's keep the, um, I'd like I like the integration points and I like the implementation status of next step.  
**Michał Skowron:** So the integration points are they even correct?  
   
 

### 00:27:05

   
**Michał Skowron:** But you first you need to you know check it for correctness.  
**Mike Clay:** Yeah, I did. Before I regenerated the document, I told it to check the existing document for correctness. Um, so we can move them. I want to keep them, but they don't necessarily need to live in the same document.  
**Michał Skowron:** But this is like weird like what is this P2P like Cardano with this functionality framework integrated what is this do you understand even this point for me is like doesn't have anything for transactions proper tagging what is this I don't understand signature integration connected to existing system Like what does it bring  
**Mike Clay:** Yeah. Yeah. Well, I need to go. Yeah. Well, I mean, I can go away and get it to explode or just remove it because if it's not if it doesn't have any utility, then it can be removed. Sorry.  
**Michał Skowron:** like when I we read this phase two it was actually useful but when I'm reading this I don't understand what and why is this Okay.  
   
 

### 00:27:59

   
**Mike Clay:** Uh yeah, that's a good it's a good question, but it doesn't necessarily mean it's saying it's wrong. Just it's just the way it's the way it's worded the way it's worded its interpretation of the integration points is not doesn't lend itself to understanding. So, what I'm tempted to do is move this to a different file and ask it to be more verbose about what each of these points mean.  
**Michał Skowron:** Yeah. But please spend some time if you want to put it you know as some reference to I really insist that you spend some time to make it shorter and to to make sure that the information is correct and I I am quite sure it's going to make the future implementations and you know everything if if you you fit it to AI it's going to be right there.  
**Mike Clay:** Yeah. Yeah. Yeah. For sure. Uh yeah. I mean, this was mainly the progress kind of report. So, um yeah, we don't really need all of this. Yeah, I mean we're we're not following this week's thing.  
   
 

### 00:29:19

   
**Mike Clay:** So it's too verbose.  
**Michał Skowron:** No. Yeah, it is.  
**Mike Clay:** It's too verbose at the moment. So I think if we can refine state next steps that will be useful. So  
**Michał Skowron:** Yeah, I think the best is like these things which to be done. It's very good and if you create tasks from that and then even put you know that if it depends how good description you will create in the task you can have then then for each task you can use AI to to make the plan more and but but what I noticed it's complicated system so usually if you just try to plan everything very detailed up front it won't be correct.  
**Mike Clay:** Yeah. No. No. It never is. It never is because it's always evolutionary. There's always an evolutionary  
**Michał Skowron:** Yes, it's always evolutionary. So you need to keep some some level of you know of of the plan and especially when it starts to create a code which it gives to the plan. This code is usually not tested.  
   
 

### 00:30:19

   
**Michał Skowron:** So it might not even compile and then if you give such code as an example for AI to implement then the code it creates is not compiling.  
**Mike Clay:** H. Yeah.  
**Michał Skowron:** it uses as a plan and it messes up everything. So what I found if you're about to put in the plan the code which is not tested not compile it's better to not put any code at all.  
**Mike Clay:** Yeah. Yeah. I mean, yeah, there's a lot we can remove from here. So, yeah.  
**Michał Skowron:** Okay. Anyway, we kind of have the plan. So you create the task and which one would you like to take?  
**Mike Clay:** Uh I don't know. Let me create the test first, but I'm I'm I'm I'd be happy to look at the RPC client elements.  
**Michał Skowron:** You don't want the witnessing.  
**Mike Clay:** I can do witnessing as well.  
**Michał Skowron:** It's It's I think it's going to be more separate from what I'm doing and it's going to move us closer to the client.  
**Mike Clay:** Uh okay.  
   
 

### 00:31:10

   
**Mike Clay:** Have you already have you already touched the client implementation then?  
**Michał Skowron:** I have to do that. But I I just use this one method. But what I think is like the client is kind of easy. I'd like to have you know uh but so I'd like I'd rather have you know witnessing from you to connect with this part I'm doing and then we will have like whatever who whoever does client is fine because  
**Mike Clay:** It is. Yeah. Relatively speaking it's easy because it's just  
**Michał Skowron:** that client is going to be uh I think you should do it the way it's done as Bitcoin because it's UTXO system you're going to  
**Mike Clay:** Yeah. Yeah. So, what I might do is um I might go away and find see if I can find any examples of Conano witnessing elsewhere. Yeah.  
**Michał Skowron:** do it almost the same I'd rather and you know it's already in the code so if you instruct AI to do it in the same way uh then it should do a good job but you need to of course understand if it's uh if it's if you did it correctly to put some time into checking that it's correct and that might be some and of course this the difference will be in the block structure because you  
   
 

### 00:32:13

   
**Mike Clay:** Yeah.  
**Michał Skowron:** would have to probably create some you know some data types. Oh but but you know but for witnessing it's not in was so you can use all those libraries like Cardano serialization leap or palace.  
**Mike Clay:** Here. Oh, pal.  
**Michał Skowron:** So I thinkization leap is more mature but it's a little bigger like paras is a little bit newer.  
**Mike Clay:** So we we use pal.  
**Michał Skowron:** I'm using paras but for for for multain engine I use I would use card realization loop you can  
**Mike Clay:** Yeah. Okay. So for the record, uh Cardano serialization lib is one possible uh support uh library for witness implementation and palace is the other.  
**Michał Skowron:** yes and palace is the other yeah so I If you use them, you it it could simplify.  
**Mike Clay:** Yeah. Yeah. Cool. Yeah. Yeah, that sounds good. Okay. Yeah, there's enough there's enough for now, I think, to be going on with. Uh, let me go away and make some tasks. Uh, yeah, I'm happy to pick up the witness implementation to start with.  
   
 

### 00:33:36

   
**Mike Clay:** Um, I left a couple of comments on your PR.  
**Michał Skowron:** Mhm.  
**Mike Clay:** So, no, no, only Yeah.  
**Michał Skowron:** Did you have any some more comments? Because I've seen like two comments. I because I I am I think I unsw Yeah, those were like small ones.  
**Mike Clay:** Yeah, nothing nothing major. I just wondered if there was some commonization available between the Yeah, it's fine.  
**Michał Skowron:** Yeah, there are. But I wouldn't touch that to be honest at this point. It's like they repeat this every in every like implementation.  
**Mike Clay:** I just thought you Yeah.  
**Michał Skowron:** So I just wouldn't you know it's like it's optimization. Mhm. And about those binary files, I think it's fine for for now especially that you need to change this journalist files. I've answered so and in this way you don't have to change this.  
**Mike Clay:** Yeah.  
**Michał Skowron:** You have you know prepared keys, prepared vault keys. So you can use them instantly.  
**Mike Clay:** Yeah. Yeah. Okay. Yeah, that's fine then. So, I'll I'll approve that one and then we can get that merged and then proceed to the next.  
**Michał Skowron:** Mhm.  
**Mike Clay:** So, yeah. Cool. All right. Well, um let's let's uh wrap up here then and I'll go and create these tasks. And what I'll do is I'll update this document as well and I'll let you know when the documents updated so you can review it.  
**Michał Skowron:** Mhm. Okay. I just try to not, you know, just get get rid of those unnecessary stuff because I'm spending if if if if there those are if there are those long documents, I'm spending too much time of checking like checking  
**Mike Clay:** Yeah.  
**Michał Skowron:** if those if those are correct really takes time. So Mhm. Mhm. You have a nice day.  
   
 

### Transcription ended after 00:35:38

*This editable transcript was computer generated and might contain errors. People can also change the text after it was created.*