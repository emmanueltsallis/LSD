# Laboratory for Simulation Development (LSD) Language Documentation

**************************************************************

	LSD 8.1 - July 2023
	written by Marco Valente, Universita' dell'Aquila
	and by Marcelo Pereira, University of Campinas

	Copyright Marco Valente and Marcelo Pereira
	LSD is distributed under the GNU General Public License

**************************************************************

LSD (Laboratory for Simulation Development) is a powerful language and platform for the creation and execution of simulation models. LSD has been conceived and developed in order to automatically manage all the technical elements required to run a simulation program. Yet, the modeler still enjoys great freedom to implement whatever computational model she desires, since the LSD core is, technically speaking, a pure C++ API (application programmer interface). This way, LSD provides the benefits of C++ (flexibility, speed, reliability, portability) while retaining the simplicity and productivity of an integrated development environment (IDE).

A LSD model is composed by its computational content, the equations, and by the model configuration. A developing environment (LSD Model Manager, LMM) permits to write the equations of the model in an easy way, expressing them like standard difference equations using a simplified macro language (a more advanced C++ interface is also available). Using LMM the equations are compiled and automatically embedded in a standalone native program, together with the required LSD libraries. The resulting LSD model program allows the definition of the model configuration and the running of the simulation using an easy-to-use graphical user interface (GUI). This way, the entire model development cycle can be managed from the LSD’s GUI, from the equation design and coding to the definition of the parameter and initial values and the presentation of the results.

Modelers and users of LSD models need to concentrate exclusively on the model, avoiding dealing with technical aspects of programming not related to the actual model. For example, a simple menu command in LMM creates a new model, and another compiles the equations, checks for errors and runs the LSD program. LSD models are endowed with advanced controls, catching errors whenever they occur and providing information on how to fix them.

Users of existing LSD models can load pre-packaged configurations of the model (e.g., initial values, number of steps) and reproduce the simulation, as suggested by the model author, without requiring any programming knowledge. New configurations are created with extremely simple and intuitive interfaces, so that even non-technical users can make use of simulation models.

LSD is particularly suited to implement agent-based models, given its embedded multi-layered, object-oriented nature. However, at its very core, LSD is nothing else than a pure, high-efficiency C++ parallel scheduling engine. However, with the added layer of graphical interfaces, any computational structure can be implemented in LSD with ease.

A model is composed by chunks of basic, macro-driven C++ code representing the (normally simple) difference equations of the model. Modelers define the equations of the model and the initial required data using simple and intuitive graphical interfaces. At the run time LSD automatically arranges the pieces of code in the correct sequence and collects the results, signaling possible inconsistencies.

A crucial characteristic of a LSD model is their inherently modularity. You can (and should) implement one piece (equation) at a time, test it, and then extend the model with a new part. LSD also automatically provides all the documentation concerning the model (e.g., which equation uses which parameter), so that even very complex models can be easily reviewed without one getting lost in the middle of messy code.

LSD appeals to novel programmers because it facilitates a gradual, trial-and-error learning process to modelling, signaling errors and suggesting solutions. However, LSD is also useful for experienced programmers because the language does not pose any limitation to the models it can implement while the platform significantly facilitates the management of complex models.

An extensive documentation covers every aspect of LSD, with tutorials, explanations of the interfaces and hints on (macro) programming.

