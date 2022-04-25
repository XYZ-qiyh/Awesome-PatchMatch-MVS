%%%%%%%%% TITLE
\title{\LaTeX\ Author Guidelines for 3DV Proceedings}

\author{First Author\\
Institution1\\
Institution1 address\\
{\tt\small firstauthor@i1.org}
% For a paper whose authors are all at the same institution,
% omit the following lines up until the closing ``}''.
% Additional authors and addresses can be added with ``\and'',
% just like the second author.
% To save space, use either the email address or home page, not both
\and
Second Author\\
Institution2\\
First line of institution2 address\\
{\tt\small secondauthor@i2.org}
}

\maketitle
% \thispagestyle{empty}

%%%%%%%%% ABSTRACT
\begin{abstract}
   The ABSTRACT is to be in fully-justified italicized text, at the top
   of the left-hand column, below the author and affiliation
   information. Use the word ``Abstract'' as the title, in 12-point
   Times, boldface type, centered relative to the column, initially
   capitalized. The abstract is to be in 10-point, single-spaced type.
   Leave two blank lines after the Abstract, then begin the main text.
   Look at previous 3DV abstracts to get a feel for style and length.
\end{abstract}

###  Patch-Match for Multi-View Stereo

\label{sec:patch-match}
The PatchMatch seminal paper by Barnes \etal \cite{barnes2009patchmatch} proposed a general method to efficiently compute an approximate nearest neighbor function defining the pixelwise correspondence among patches of two images. 
The idea is to use a collaborative search which exploits local coherency. 
PatchMatch initializes each pixel of an image with a random guess about the location of the nearest neighbor in the second image. 
Then, each pixel propagates its estimate to the neighboring pixels and, among these estimates, the most likely is assigned to the pixel itself. 
As a result the best estimates spread along the entire image.

Bleyer \etal \cite{bleyer2011patchmatch} re-framed this method into the stereo matching realm. Indeed, for each image patch, stereo matching looks in the second image for the corresponding patch, \ie the nearest neighbor in the sense of photometric consistency. 
To improve its robustness the matching function is not limited to fixed sized squared windows, but it extends PatchMatch to estimate a pixel-wise plane orientation adopted to define the matching procedure on slanted support windows.
Heise \etal \cite{heise2013pm} integrated the PatchMatch for stereo into a variational formulation to regularize the estimate with quadratic relaxation. This approach produces smoother depth estimates while preserving edges discontinuities.

The previous works successfully applied the PatchMatch idea to the pair-wise stereo matching problem. The natural extension to Multi-View Stereo was proposed by Shen \cite{shen2013accurate}. Here the author selects a subset of camera pairs depending on the number of shared points computed by Structure from Motion and their mutual parallax angle. Then he estimates a depth map for the selected subset of camera pairs through a simplified version of the method of Bleyer \etal \cite{bleyer2011patchmatch}.
The algorithm refines the depth maps by enforcing consistency among multiple views, and it finally merges the depth maps into a point cloud.

A different multi-view approach by Galliani \etal \cite{Galliani_2015_ICCV} modifies the PatchMatch propagation scheme in such a way that computation can better exploit the parallelization of GPUs. Differently, from Shen \cite{shen2013accurate}, they aggregate, for each reference camera, a set of matching costs compute from different source images.
One of the major drawbacks of these approaches is the decoupled depth estimation and camera pairs selection.
Xu and Tao \cite{xu2018multi} recently proposed an attempt to overcome this issue; they extended \cite{Galliani_2015_ICCV} with a more efficient propagation pattern and, in particular, their optimization procedure jointly considers all the views and all the depth hypotheses.


Rather than considering the whole set of images to compute the matching costs, Zheng \etal \cite{zheng14joint} proposed an elegant method to deal with view selection. They designed a robust method framing the joint depth estimation and pixel-wise view selection problem into a variational approximation framework. Following a generalized Expectation Maximization paradigm, they alternate depth update with a PatchMatch propagation scheme, keeping the view selection fixed, and pixel-wise view inference with the forward-backward algorithm, keeping the depth fixed.


Sch{\"o}nberger \etal \cite{schonberger2016pixelwise} extended this method to jointly estimate per-pixel depths and normals, such that,  differently from \cite{zheng14joint}, the knowledge of the normals enables slanted support windows to avoid the fronto-parallel assumption. Then they add view-dependent priors to select views that more likely induce robust matching cost computation.

The PatchMatch based methods described thus far, have been proven to be among the top performing approachs in several MVS benchmarks \cite{seitz_et_al06,strecha2008,jensen2014large,schoeps2017cvpr}. However, some issues are still open. In particular, most of them strongly rely on photo-consistency measures to discriminate among depth hypotheses. Even if this works remarkably for textured areas and the propagation scheme partially induces smoothness, untextured regions are often poorly reconstructed. 
For this reason, we propose two proxies to improve the reconstruction where untextured areas appear. On the one hand, we seamlessly extend the probabilistic framework to explicitly detect and handle untextured regions by extending the set of PatchMatch hypotheses. On the other side, we complete the depth estimation with a refinement procedure to fill the missing depth estimates.

%-------------------------------------------------------------------------
###  Review of the COLMAP framework

\label{sec:review}
In this section we review the state-of-the-art framework proposed by Sch{\"o}nberger \etal \cite{schonberger2016pixelwise} which builds on top of the method presented by Zheng \etal \cite{zheng14joint}.
Let note that in the following, we express the coordinate of the pixel only with a value $l$, since both frameworks sweep independently every single line of the image alternating between rows and columns.

Given a reference image $\mathbf{X} ^ { \mathrm{ ref } }$ and a set of source images $\mathbf{X} ^ { \mathrm{ src } } = \left\{ X ^ { m } | m = 1 \ldots M \right\}$, the framework estimates  the depth $\theta_l$ and the normal $\mathbf{n}_l$ of each pixel $l$, together with a binary variable $Z_l^m\in\{0,1\}$, which indicates if $l$ is visible in image $m$. 
This is framed into a Maximum-A Posteriori (MAP) estimation where the posterior probability is:
{\scriptsize
\begin{multline}
\label{eq:MAP}
P(\mathbf{Z} , \mathbf{ \theta } , \mathbf { N } | \mathbf { X } )  = \frac{ P (\mathbf{Z} , \mathbf{ \theta } , \mathbf { N } , \mathbf { X } )}{P(\mathbf{X})}=\\
=\frac{ 1}{P(\mathbf{X})}
\prod_{l=1}^{L}\prod_{m=1}^{M}\left[P\left(Z_{l,t}^{m}|Z_{l-1,t}^{m},Z_{l,t-1}^{m}\right)\right.\\
\left.P\left(X_{l}^{m}|Z_{l}^{m},\theta_{l},\mathbf{n}_{l},X^{ref}\right)P\left(\theta_{l},\mathbf{n}_{l}|\theta_{l}^{m},\mathbf{n}_{l}^{m}\right)\right],
\end{multline}}
where $L$ is the number of pixels considered in the current line sweep,
$\mathbf { X } = \left\{ \mathbf{X}^{\mathrm{ src }}, \mathrm{X}^{\mathrm{ref}}\right\}$ and $\mathbf{N}=\left\{\mathbf{n}_{l} | l = 1 \ldots L \right\}$.
The likelihood term 
\begin{equation}
\label{eq:like}\scriptsize
P\left(X_{l}^{m}|Z_{l}^{m},\theta_{l}\right)=
\left\{\begin{array}{ll}{\frac{1}{NA} \exp \left(-\frac{\left(1-\rho_{l}^{m}\left(\theta_{l}\right)\right)^{2}}{2\sigma_{\rho}^{2}}\right)} & {\text{if } Z_{l}^{m}=1}\\{\frac{1}{N}\mathcal{U}}&{\text{if } Z_{l}^{m}=0,}\end{array}\right.
\end{equation}
represents the photometric consistency of the patch $X_{l}^{m}$, which belongs to a non-occluded source image $m$ and is around the pixel corresponding to the point at $l$, with respect to the patch $X_{l}^{ref}$ around $l$ in the reference image.
The photometric consistency $\rho$ is computed as a bilaterally weighted NCC, $A = \int_{- 1 } ^ { 1 } \exp \left\{ - \frac { ( 1 - \rho ) ^ { 2 } } { 2 \sigma_{\rho } ^ { 2 } } \right\} d \rho$ and the constant $N$ cancels out in the optimization.
The likelihood term $P\left(\theta_{l},\mathbf{n}_{l}|\theta_{l}^{m},\mathbf{n}_{l}^{m}\right)$ represents the geometric consistency and enforces multi-view depth and normal coherence.
Finally $P\left(Z_{l,t}^{m}|Z_{l-1,t}^{m},Z_{l,t-1}^{m}\right)$ favors image occlusion indicators which are smooth both spatially and along the successive iteration of the optimization procedure.

Being Equation \eqref{eq:MAP} intractable, Zheng \etal \cite{zheng14joint} proposed to use variational inference to approximate the real posterior with a function $q(\mathbf{Z},\mathbf{\theta}, \mathbf{N})$ such that the KL divergence of the two functions is minimized.
Sch{\"o}nberger \etal \cite{schonberger2016pixelwise} factorize  $q(\mathbf{Z},\mathbf{\theta}, \mathbf{N})=q(\mathbf{Z})q(\mathbf{\theta}, \mathbf{N})$ and, to estimate such approximation, they propose a variant of the Generalized Expectation-Maximization algorithm \cite{neal1998view}.
In the E step, the values $(\mathbf{\theta}, \mathbf{N})$ are kept fixed, and, in the resulting Hidden Markov Model, the function $q(Z_{l,t}^m)$ is computed by means of message passing. 
In the M step, viceversa, the values of $Z_{l,t}^m$ are fixed, the function $q(\mathbf{\theta}, \mathbf{N})$ is constrained to the family of Kroneker delta functions $q({\theta_l}, \mathbf{n}_l)=q(\theta_l=\theta_l^*, \mathbf{n}_l^*)$.
The new optimal values of $\mathbf{\theta}_l$ and $\mathbf{N}_l$ are computed as:
% \begin{equation}
%     \left(\theta_{l }^{\mathrm{opt}}, \mathbf {n}^{\mathrm{opt}}\right) = \underset{\theta_{l}^{*}}{\operatorname{argmin}} \sum_{m = 1}^{M} P_{l}( m ) \left( 1 - \rho_{l }^{m} \left( \theta_{l }^{*} , \mathbf { n }_{l } ^ { * }\right) \right)
% \end{equation}
\begin{equation}
\scriptsize
\label{eq:optim_depth_norm}
    \left( \hat { \theta }_{l } ^ { \text { opt } } , \hat { \mathbf { n } }_{l } ^ { \mathrm{ opt } } \right) = \underset { \theta_{l } ^ { * } , \mathbf { n }_{l } ^ { * } } { \operatorname { argmin } } \frac { 1 } { | S | } \sum_{m \in S } \left( 1 - \rho_{l } ^ { m } \left( \theta_{l } ^ { * } , \mathbf { n }_{l } ^ { * } \right) \right),
\end{equation}
where $S$ is a subset of sources images, randomly sampled according to a probability $P_l(m)$. Probability  $P_l(m)$ favors images not occluded, and coherent with  three priors which encourage good inter-cameras parallax, similar resolution and camera, front-facing the 3D point defined by ${ \theta }_{ l }^{ * } , \mathbf{n}_{l}^{*}$.

According to the PatchMatch scheme proposed in \cite{schonberger2016pixelwise}, the pair $ \left( \theta_{l } ^ { * } , \mathbf { n }_{l } ^ { * } \right)$ evaluated in Equation \eqref{eq:optim_depth_norm} is chosen among the following set of hypotheses:
{\footnotesize
\begin{multline}
\label{eq:hypotheses}
    \left\{ \left( \theta_{l }, \mathbf{ n }_{l } \right) , \left( \theta_{l - 1}^{\mathrm{ prp } }, \mathbf{ n }_{l - 1 } \right) , \left( \theta_{l}^{\mathrm{ rnd } }, \mathbf{ n }_{l } \right) , \left( \theta_{l }, \mathbf{ n }_{l}^{\mathrm{ rnd } } \right) ,\right. \\\\
    \left. \left( \theta_{l}^{\mathrm{ rnd } }, \mathbf{ n }_{l}^{\mathrm{ rnd } } \right) , \left( \theta_{l}^{\mathrm{ prt } }, \mathbf{ n }_{l } \right) , \left( \theta_{l }, \mathbf{ n }_{l}^{\mathrm{ prt } } \right) \right\},
\end{multline}}
where $\left( \theta_{l}, \mathbf{ n }_{l} \right)$ comes from the previous iteration,  $\left( \theta_{l-1}, \mathbf{ n }_{l-1} \right)$ is the estimate from the previous pixel of the scan, $\left( \theta_{l}^{\mathrm{ rnd}}, \mathbf{ n }_{l} \right)$ is a random hypothesis and finally, $\theta_{l}^{\mathrm{prt}}$ and $ \mathbf{n}_{l}^{\mathrm{prt}}$ are two small perturbations of the estimates $ \theta_{l}$ and $\mathbf{n}_{l}$.