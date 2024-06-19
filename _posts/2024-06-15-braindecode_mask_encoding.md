---
title: Contribution to Braindecode - Add new augmentation method (MaskEncoding)
date: 2024-06-15
categories: [Braindecode, Open-Science, MAC5856 - Desenvolvimento de Software Livre]
tags: [open source, open science]     # TAG names should always be lowercase
comments: false
math: true
---

Comments on my contribution to braindecode, adding a new augmentation method called MaskEncoding.

A few weeks ago I proposed a new data augmentation technique for electroencephalography (EEG) decoding to [Braindecode](https://braindecode.org/stable/index.html), named [SegmentationReconstruction](https://gustavohenriquesr.github.io/posts/braindecode_augmentation/). It was my first contribution to the library and the mantainers seem satisfied with the addition of new methods, as they are in the process of adding new features to soon release a new updated version. In this sense, one of the mantainers of Braindecode opened an issue about adding more data augmentation techniques e tagged me to work with it, since I recently made a related contribuition and showed interest in improving the library. Therefore, I looked up for data augmentation techniques proposed in the field of BCI (Brain-Computer Interface) decoding, as well as in the field of computer vision, in which we can sometimes adapt the method according to the data type. So, I founded a technique proposed in this year (2024) for steady-state visual evoked potential (SSVEP)-based brain-computer interface (BCI) system, which I thought I could adapt to the context of EEG in Braindecode. The [article](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10439237) proposes the use of mask encoding to generate new artificial trials and I implemented a inspired version of it, named MaskEncoding, adding new features to improve the quality of the synthetic data.

## About data augmentation

The Mask encoding technique proposed in the article is based on the following. A random continuous segment is chosen as the masking window, and the value of the EEG data in this window is set to 0. Thus, this process forces the model (network architecture) to extract only the most important features of the signal, and not how it behaves at all times (which could lead to overfitting). In this way, the model better understands EEG patterns rather than memorizing all the data, improving its generalization capacity.  

![Data Augmentation](assets/img/mask_encoding.png)
_EEG-ME for one electrode channel data of a sample. From the paper proposing mask encoding._

### My modifications
<!--
Uma vez que a ideia de masking é simple e se assemelha a um method já disponível no Braindecode, chamado SmoothTimeMask, decidi adicionar alguns parâmetros extras além da proposta inicial. Diferente do SmoothTimeMask, no qual o sinal vai diminuindo aos poucos até ser zerado, o MaskEncoding é uma versão mais bruta, que zera os valores do sinal sem suavização na janela de ocultação. Assim, como forma de melhorar o uso dessa aumentação e torná-la mais relevante, decidi adicionar dois parâmetros que o usuário é capaz de configurar. O primeiro parâmetro envolve escolher uma razão de ocultação dos dados, que varia de 0 (nenhuma ocultação) até 1 (um sinal flat zerado). Já o segundo parâmetro permite com que, dada a razão de ocultação, o usuário possa criar mais de uma janela de ocultação, a depender do número de divisões que é fornecido. Assim, as ocultações não precisam estar localizadas em um região específica do sinal a cada transformação, mas pode estar espalhada, o que pode contribuir para extração de características mais robustas pelo modelo em relação à proposta inicial.
-->

Since the idea of masking is simple and resembles a method already available in Braindecode, called SmoothTimeMask, I decided to add some extra parameters beyond the initial proposal. Unlike SmoothTimeMask, in which the signal gradually decreases until it is zeroed out, MaskEncoding is a rougher version, which zeroes out the signal values right away in the masking window. So, as a form of improving the use of this augmentation and making it more relevant, I decided to add two parameters that the user is able to configure. The first parameter involves choosing a mask ratio, which ranges from 0 (no mask) to 1 (a zeroed flat signal). The second parameter allows the user to create more than one masking window, depending on the number of splits provided, given the masking ratio. Thus, the maskings do not have to be located in a specific region of the signal for each transformation, but can be spread out, which can contribute to the extraction of more robust characteristics by the model compared to the initial proposal.

## Process of integration in Braindecode
<!--
Como eu já tinha feito uma contribuição similar anteriormente no repositório, fiz a abertura de um [Pull Request](https://github.com/braindecode/braindecode/pull/631) com todos os arquivos necessários atualizados para assimilação da técnica na biblioteca. Assim, como minha implementação em boa parte já estava no formato de preferência dos mantenedores e com os testes de integração já feitos, poucas foram as mudanças sugeridas na primeira revisão, contemplando somente nomeação de funções e variáveis. Com as mudanças realizadas, um dos mantenedores manifestou que a implementação estava boa para merge mas antes gostaria que eu fizesse alguns testes com os datasets disponíveis de EEG para verificar a efetividade do uso do MaskEncoding no treino de classificação de dados de EEG. Assim, eu segui para fazer os testes. Eu utilizei o dataset BCI Competition IV 2a e fiz uma busca de parâmetros tanto para mask ratio quanto para o número de splits, e obtive os resultados que seguem nos dois gráficos abaixo.

O primeiro, no qual testamos a melhora da acurácia em relação a nenhuma trasnformação no treino mostra que o uso de mais de um segmento de masking é melhor para o modelo na classificação. Já no segundo gráfico vemos que razões de masking mais altas que 10% quando utilizado somento um segmento pioram o desempenho do modelo. Mesmo com os resultados demonstrando que o uso da aumentação pode tanto piorar quanto melhorar o desempenho, a conclusão que podemos tirar até o momento é que para utilizar esse técnica é necessário algum tipo de fine-tuning para encontrar os melhores parâmetro a depender da tarefa sendo realizada. Os gráfico foram apresentados no PR e ainda estou na espera de um feedback por parte dos mantenedores.
-->

As I had already made a similar contribution to the repository, I opened a [Pull Request](https://github.com/braindecode/braindecode/pull/631) with all the necessary files updated to assimilate the technique into the library. So, as my implementation was largely already in the format preferred by the maintainers and with the integration tests already done, there were few changes suggested in the first review, just naming functions and variables. With the changes made, one of the maintainers said that the implementation was fine for merge, but first he would like me to do some tests with the available EEG datasets to check the effectiveness of using MaskEncoding in EEG data classification training. So I proceeded to do the tests. I used the BCI Competition IV 2a dataset and searched for parameters for both the mask ratio and the number of splits, and obtained the results shown in the two graphs below.

![Test splits](assets/img/splits.png)
_Split parameter selection on the BCI IV 2a dataset. Models were trained between the range of 1 to 5 splits. Red dashed line corresponds to model trained without data augmentation._

![Test mask ratio](assets/img/mask_ratio.png)
_Mask ratio parameter selection on the BCI IV 2a dataset. Models were trained between the range of 0.1 to 0.5 splits, with 10 points linearly spaced. Red dashed line corresponds to model trained without data augmentation._

The first plot, in which we test the improvement in accuracy compared to no training transformation, shows that using more than one masking segment is better for the model in classification. In the second graph, we see that masking ratios higher than 10% when only one segment is used worsen the model's performance. I decided then to make the same search for mask ratio but using 5 segments instead of just 1. The graph below show an improvement using the data augmentation when mask ratio is up to 35% approximately.

![Test mask ratio 5 seg](assets/img/mask_ratio_5.png)
_Mask ratio parameter selection on the BCI IV 2a dataset. Models were trained between the range of 0.1 to 0.5 splits, with 10 points linearly spaced, and using 5 segmentations of masking window. Red dashed line corresponds to model trained without data augmentation._

Even though the results show that the use of augmentation can both worsen and improve performance, the conclusion we can draw so far is that using this technique requires some kind of fine-tuning to find the best parameters depending on the task being performed. The graphs were presented at PR and the augmentation technique has been accepted to integrate Braindecode. You can found the documentation [here](https://braindecode.org/dev/generated/braindecode.augmentation.MaskEncoding.html). 