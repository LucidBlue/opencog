INCLUDE_DIRECTORIES (${GUILE_INCLUDE_DIRS})

IF (HAVE_CYTHON)
	INCLUDE_DIRECTORIES (${PYTHON_INCLUDE_DIRS})
ENDIF (HAVE_CYTHON)


ADD_LIBRARY (builtinreqs SHARED
	BuiltinRequestsModule
	Request
	SleepRequest
	LoadModuleRequest
	ListRequest
	ShutdownRequest
	UnloadModuleRequest
	ListModulesRequest
)

TARGET_LINK_LIBRARIES(builtinreqs
	${ATOMSPACE_LIBRARY}
	${COGUTIL_LIBRARY}
)

INSTALL (TARGETS builtinreqs
	DESTINATION "lib${LIB_DIR_SUFFIX}/opencog/modules")

# ------------------------------------------------------------

ADD_LIBRARY (server SHARED
	Agent
	BaseServer
	CogServer
	Request
	NetworkServer
	ServerSocket
	ConsoleSocket
	SystemActivityTable
)

TARGET_LINK_LIBRARIES(server
	nlp-types
	${ATOMSPACE_LIBRARY}
	${Boost_FILESYSTEM_LIBRARY}
	${Boost_THREAD_LIBRARY}
	dl
)

IF (HAVE_CYTHON)
	TARGET_LINK_LIBRARIES(server
		${ATOMSPACE_PythonEval_LIBRARY}
		py-shell ${ATOMSPACE_LIBRARIES} ${PYTHON_LIBRARIES})
ENDIF (HAVE_CYTHON)

IF (HAVE_GUILE)
	TARGET_LINK_LIBRARIES(server scheme-shell ${ATOMSPACE_LIBRARIES})
ENDIF (HAVE_GUILE)

IF (ODBC_FOUND)
	TARGET_LINK_LIBRARIES(server PersistModule)
ENDIF (ODBC_FOUND)

# ------------------------------------------------------------
# This defines the guile scheme loadable module

IF (HAVE_GUILE)
	ADD_LIBRARY (guile-cogserver SHARED
		CogServerSCM
	)

	TARGET_LINK_LIBRARIES(guile-cogserver
		server
		${Boost_SYSTEM_LIBRARY}
		${GUILE_LIBRARIES}
	)

	INSTALL (TARGETS guile-cogserver DESTINATION "lib${LIB_DIR_SUFFIX}/opencog")
ENDIF (HAVE_GUILE)

# ------------------------------------------------------------

ADD_EXECUTABLE(cogserver
	CogServerMain
)

TARGET_LINK_LIBRARIES(cogserver
	server
	${Boost_SYSTEM_LIBRARY}
	${GUILE_LIBRARIES}
)

SET_TARGET_PROPERTIES(cogserver PROPERTIES LINK_DIRECTORIES "${BOOST_LIBRARY_DIRS}")
IF (WIN32)
	SET_TARGET_PROPERTIES(cogserver PROPERTIES LINK_FLAGS "/NODEFAULTLIB:libctm")
ENDIF (WIN32)

INSTALL (TARGETS cogserver RUNTIME DESTINATION "bin")

INSTALL (TARGETS server DESTINATION "lib${LIB_DIR_SUFFIX}/opencog")

INSTALL (FILES
	Agent.h
	BaseServer.h
	BuiltinRequestsModule.h
	CogServer.h
	ConsoleSocket.h
	Factory.h
	IHasMimeType.h
	IRequestComplete.h
	ListRequest.h
	LoadModuleRequest.h
	Module.h
	NetworkServer.h
	SocketListener.h
	SocketPort.h
	SystemActivityTable.h
	Registry.h
	Request.h
	RequestClassInfo.h
	RequestResult.h
	ShutdownRequest.h
	UnloadModuleRequest.h
	DESTINATION "include/${PROJECT_NAME}/server"
)
