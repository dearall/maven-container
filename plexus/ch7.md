

```java
public interface PlexusContainer
{
    String ROLE = PlexusContainer.class.getName();

    // ------------------------------------------------------------------------
    // Lookup
    // ------------------------------------------------------------------------

    /**
     * Looks up and returns a component object with the given unique key or role.
     * @param role a unique key for the desired component
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    Object lookup( String role )
        throws ComponentLookupException;

    /**
     * Looks up and returns a component object with the given unique role/role-hint combination.
     * @param role a non-unique key for the desired component
     * @param roleHint a hint for the desired component implementation
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    Object lookup( String role, String roleHint )
        throws ComponentLookupException;

    /**
     * Looks up and returns a component object with the given unique key or role.
     * @param type the unique type of the component within the container
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( Class<T> type )
        throws ComponentLookupException;

    /**
     * Looks up and returns a component object with the given unique role/role-hint combination.
     * @param type the non-unique type of the component
     * @param roleHint a hint for the desired component implementation
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( Class<T> type, String roleHint )
        throws ComponentLookupException;

    /**
     * Looks up and returns a component object with the given unique role/role-hint combination.
     * @param type the non-unique type of the component
     * @param role a non-unique key for the desired component
     * @param roleHint a hint for the desired component implementation
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( Class<T> type, String role, String roleHint )
        throws ComponentLookupException;

    /**
     * Looks up and returns a component object matching the given component descriptor.
     * @param componentDescriptor the descriptor of the component
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( ComponentDescriptor<T> componentDescriptor )
        throws ComponentLookupException;

    /**
     * Looks up and returns a List of component objects with the given role.
     * @param role a non-unique key for the desired components
     * @return a List of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    List<Object> lookupList( String role )
        throws ComponentLookupException;

    /**
     * Looks up and returns a List of component objects with the given role.
     * @param role a non-unique key for the desired components
     * @param roleHints the list of hints.
     * @return a List of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    List<Object> lookupList( String role, List<String> roleHints )
        throws ComponentLookupException;

    /**
     * Looks up and returns a List of component objects with the given role.
     * @param type the non-unique type of the components
     * @param <T> The type.
     * @return a List of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> List<T> lookupList( Class<T> type )
        throws ComponentLookupException;

    /**
     * Looks up and returns a List of component objects with the given role.
     * @param type the non-unique type of the components
     * @param roleHints the list of hints.
     * @param <T> The type.
     * @return a List of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> List<T> lookupList( Class<T> type, List<String> roleHints )
        throws ComponentLookupException;

    /**
     * Looks up and returns a Map of component objects with the given role, keyed by all available role-hints.
     * @param role a non-unique key for the desired components
     * @return a Map of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    Map<String, Object> lookupMap( String role )
        throws ComponentLookupException;

    /**
     * Looks up and returns a Map of component objects with the given role, keyed by all available role-hints.
     * @param role a non-unique key for the desired components
     * @param roleHints the list of hints.
     * @return a Map of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    Map<String, Object> lookupMap( String role, List<String> roleHints )
        throws ComponentLookupException;

    /**
     * Looks up and returns a Map of component objects with the given role, keyed by all available role-hints.
     * @param type the non-unique type of the components
     * @param <T> The type.
     * @return a Map of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> Map<String, T> lookupMap( Class<T> type )
        throws ComponentLookupException;

    /**
     * Looks up and returns a Map of component objects with the given role, keyed by all available role-hints.
     * @param type the non-unique type of the components
     * @param roleHints the list of hints.
     * @param <T> The type.
     * @return a Map of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> Map<String, T> lookupMap( Class<T> type, List<String> roleHints )
        throws ComponentLookupException;

    // ----------------------------------------------------------------------
    // Component Descriptor Lookup
    // ----------------------------------------------------------------------

    /**
     * Returns the ComponentDescriptor with the given component role and the default role hint.
     * Searches up the hierarchy until one is found, null if none is found.
     * @param role a unique role for the desired component's descriptor
     * @return the ComponentDescriptor with the given component role
     */
    ComponentDescriptor<?> getComponentDescriptor( String role );

    /**
     * Returns the ComponentDescriptor with the given component role and hint.
     * Searches up the hierarchy until one is found, null if none is found.
     * @param role a unique role for the desired component's descriptor
     * @param roleHint a hint showing which implementation should be used
     * @return the ComponentDescriptor with the given component role
     */
    ComponentDescriptor<?> getComponentDescriptor( String role, String roleHint );

    /**
     * Returns the ComponentDescriptor with the given component role and hint.
     * Searches up the hierarchy until one is found, null if none is found.
     * @param type the Java type of the desired component
     * @param role a unique role for the desired component's descriptor
     * @param roleHint a hint showing which implementation should be used
     * @param <T> The type.
     * @return the ComponentDescriptor with the given component role
     */
    <T> ComponentDescriptor<T> getComponentDescriptor( Class<T> type, String role, String roleHint );

    /**
     * Returns a Map of ComponentDescriptors with the given role, keyed by role-hint. Searches up the hierarchy until
     * all are found, an empty Map if none are found.
     * @param role a non-unique key for the desired components
     * @return a Map of component descriptors keyed by role-hint
     */
    Map<String, ComponentDescriptor<?>> getComponentDescriptorMap( String role );

    /**
     * Returns a Map of ComponentDescriptors with the given role, keyed by role-hint. Searches up the hierarchy until
     * all are found, an empty Map if none are found.
     * @param type the Java type of the desired components
     * @param role a non-unique key for the desired components
     * @param <T> The type.
     * @return a Map of component descriptors keyed by role-hint
     */
    <T> Map<String, ComponentDescriptor<T>> getComponentDescriptorMap( Class<T> type, String role );

    /**
     * Returns a List of ComponentDescriptors with the given role. Searches up the hierarchy until all are found, an
     * empty List if none are found.
     * @param role a non-unique key for the desired components
     * @return a List of component descriptors
     */
    List<ComponentDescriptor<?>> getComponentDescriptorList( String role );

    /**
     * Returns a List of ComponentDescriptors with the given role. Searches up the hierarchy until all are found, an
     * empty List if none are found.
     * @param type the Java type of the desired components
     * @param role a non-unique key for the desired components
     * @param <T> The type.
     * @return a List of component descriptors
     */
    <T> List<ComponentDescriptor<T>> getComponentDescriptorList( Class<T> type,  String role );

    /**
     * Adds a component descriptor to this container. componentDescriptor should have realmId set.
     * @param componentDescriptor {@link ComponentDescriptor}
     * @throws CycleDetectedInComponentGraphException In case of an error.
     */
    void addComponentDescriptor( ComponentDescriptor<?> componentDescriptor )
        throws CycleDetectedInComponentGraphException;

    /**
     * Releases the component from the container. This is dependent upon how the implementation manages the component,
     * but usually enacts some standard lifecycle shutdown procedure on the component. In every case, the component is
     * no longer accessible from the container (unless another is created).
     * @param component the plexus component object to release
     * @throws ComponentLifecycleException in case of an error.
     */
    void release( Object component )
        throws ComponentLifecycleException;

    /**
     * Releases all Mapped component values from the container.
     * @see PlexusContainer#release( Object component )
     * @param components Map of plexus component objects to release
     * @throws ComponentLifecycleException in case of an error.
     */
    void releaseAll( Map<String, ?> components )
        throws ComponentLifecycleException;

    /**
     * Releases all Listed components from the container.
     * @see PlexusContainer#release( Object component )
     * @param components List of plexus component objects to release
     * @throws ComponentLifecycleException in case of an error.
     */
    void releaseAll( List<?> components )
        throws ComponentLifecycleException;

    /**
     * Returns true if this container has the keyed component.
     * @param role a non-unique key for the desired component
     * @return true if this container has the keyed component
     */
    boolean hasComponent( String role );

    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param role a non-unique key for the desired component
     * @param roleHint a hint for the desired component implementation
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( String role, String roleHint );

    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param type the non-unique type of the component
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( Class<?> type );

    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param type the non-unique type of the component
     * @param roleHint a hint for the desired component implementation
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( Class<?> type, String roleHint );

    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param type the non-unique type of the component
     * @param role a non-unique key for the desired component
     * @param roleHint a hint for the desired component implementation
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( Class<?> type, String role, String roleHint );

    /**
     * Disposes of this container, which in turn disposes all of it's components. This container should also remove
     * itself from the container hierarchy.
     */
    void dispose();

    // ----------------------------------------------------------------------
    // Context
    // ----------------------------------------------------------------------

    /**
     * Add a key/value pair to this container's Context.
     * @param key any unique object valid to the Context's implementation
     * @param value any object valid to the Context's implementation
     */
    void addContextValue( Object key, Object value );

    /**
     * Returns this container's context. A Context is a simple data store used to hold values which may alter the
     * execution of the Container.
     * @return this container's context.
     */
    Context getContext();

    /**
     * Returns the Classworld's ClassRealm of this Container, which acts as the default parent for all contained
     * components.
     * @return the ClassRealm of this Container
     */
    ClassRealm getContainerRealm();

    // ----------------------------------------------------------------------
    // Discovery
    // ----------------------------------------------------------------------

    /**
     * Adds the listener to this container. ComponentDiscoveryListeners have the ability to respond to various
     * ComponentDiscoverer events.
     * @param listener A listener which responds to different ComponentDiscoveryEvents
     */
    void registerComponentDiscoveryListener( ComponentDiscoveryListener listener );

    /**
     * Removes the listener from this container.
     * @param listener A listener to remove
     */
    void removeComponentDiscoveryListener( ComponentDiscoveryListener listener );

    /**
     * Discovers components in the given realm.
     * @param childRealm {@link ClassRealm}
     * @return list {@link ComponentDescriptor}
     * @throws PlexusConfigurationException in case of an error.
     * @throws CycleDetectedInComponentGraphException in case of an error.
     */
    List<ComponentDescriptor<?>> discoverComponents( ClassRealm childRealm )
        throws PlexusConfigurationException, CycleDetectedInComponentGraphException;

    /**
     * Discovers components in the given realm.
     * @param realm the {@link ClassRealm}.
     * @param data The data.
     * @return list {@link ComponentDescriptor}
     * @throws PlexusConfigurationException in case of an error.
     * @throws CycleDetectedInComponentGraphException in case of an error.
     */
    List<ComponentDescriptor<?>> discoverComponents( ClassRealm realm, Object data )
        throws PlexusConfigurationException, CycleDetectedInComponentGraphException;

    // ----------------------------------------------------------------------------
    // Component/Plugin ClassRealm creation
    // ----------------------------------------------------------------------------

    ClassRealm createChildRealm( String id );

    ClassRealm getComponentRealm( String realmId );

    /**
     * Dissociate the realm with the specified id from the container. This will
     * remove all components contained in the realm from the component repository.
     *
     * @param componentRealm Realm to remove from the container.
     * @throws PlexusContainerException {@link PlexusContainerException}.
     */
    void removeComponentRealm( ClassRealm componentRealm )
        throws PlexusContainerException;

    /**
     * Returns the lookup realm for this container, which is either
     * the container realm or the realm set by {@link MutablePlexusContainer#setLookupRealm(ClassRealm)}.
     * @return {@link ClassRealm}
     */
    ClassRealm getLookupRealm();

    /**
     * Sets the lookup realm to use for lookup calls that don't have a ClassRealm parameter.
     * @param realm the new realm to use.
     * @return The previous lookup realm. It is advised to set it back once the old-style lookups have completed.
     */
    ClassRealm setLookupRealm(ClassRealm realm);

    /**
     * XXX ideally i'd like to place this in a plexus container specific utility class.
     *
     * Utility method to retrieve the lookup realm for a component instance.
     * If the component's classloader is a ClassRealm, that realm is returned,
     * otherwise the result of getLookupRealm is returned.
     * @param component The component.
     * @return {@link ClassRealm}
     */
    ClassRealm getLookupRealm( Object component );

    void addComponent( Object component, String role )
        throws CycleDetectedInComponentGraphException;

    /**
     * Adds live component instance to this container.
     *
     * Component instance is not associated with any class realm and will
     * be ignored during lookup is lookup realm is provided using thread context
     * classloader.
     * @param component The component.
     * @param role The role.
     * @param roleHint The hint.
     * @param <T> The type.
     */
    <T> void addComponent( T component, Class<?> role, String roleHint );
}
```