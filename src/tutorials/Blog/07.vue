<template>
  <Lesson
    :text="text"
    :code="code"
    :validate="validate"
    :modules="modules"
    :exercise="exercise"
    :solution="solution"
    :overrideErrors="true"
    lessonTitle="Traverse through all posts, starting with the most recent" />
</template>

<script>
import Lesson from '../../components/Lesson'
import text from './07.md'
import exercise from './07-exercise.md'
import shallowEqualArrays from 'shallow-equal/arrays'
import CID from 'cids'

const validate = async (result, ipfs) => {
  if (!result) {
    return { fail: 'No result was returned. Did you forget to return a result from your `traversePosts` function? Or perhaps you accidentally edited the `run` function?' }
  }

  if (result.error && result.error.message === `Cannot read property 'prev' of undefined`) {
    return { fail: 'Cannot read property `prev` of undefined. Did you try to access the value of `ipfs.dag.get()` before the function completed?' }
  }

  if (result.error && result.error.message === `Cannot read property 'value' of undefined`) {
    return { fail: 'Cannot read property `value` of undefined. Did you try to access the value of `ipfs.dag.get()` before the function completed?' }
  }

  if (result.error) {
    return { error: result.error }
  }

  if (!Array.isArray(result)) {
    return { fail: 'The return value of your traversePosts function needs to be an array.' }
  }

  const dogPostCid = 'zdpuAxe3g8XBLrqbp3NrjaiBLTrXjJ3SJymePGutsRRMrhAKS'
  const computerPostCid = 'zdpuAwwT4kGJxT7mgVZRgvmV3ke8qGNZGLuCgLhJsdBSQGM44'
  const treePostCid = 'zdpuAri55PR9iW239ahcbnfkFU2TVyD5iLmqEFmwY634KZAJV'

  try {
    if (result.length !== 3 || result === undefined) {
      return { fail: 'Your traversePosts function needs to return 3 CIDs' }
    }
    const isCids = result.every(CID.isCID)
    if (!isCids) {
      return { fail: 'Your traversePosts function needs to return CIDs.' }
    }
    const expectedCids = [treePostCid, computerPostCid, dogPostCid]
    const returnedCids = result.map(item => item.toBaseEncodedString())
    if (!shallowEqualArrays(returnedCids.sort(), expectedCids.sort())) {
      return {
        fail: 'The CIDs returned by the traversePosts function did not match the expected CIDs.',
        log: {
          returnedCids: returnedCids,
          expectedCids: expectedCids
        }
      }
    }
  } catch (err) {
    return { fail: `Your function threw an error: ${err}.` }
  }
  return { success: 'Great job! You\'ve completed this series of lessons!' }
}

const code = `/* globals ipfs */

const traversePosts = async (cid) => {
  // Your code goes here
}

const run = async () => {
  const natCid = await ipfs.dag.put({ author: "Nat" })
  const samCid = await ipfs.dag.put({ author: "Sam" })

  const treePostCid = await ipfs.dag.put({
    content: "trees",
    author: samCid,
    tags: ["outdoor", "hobby"]
  })
  const computerPostCid = await ipfs.dag.put({
    content: "computers",
    author: natCid,
    tags: ["hobby"],
    prev: treePostCid
  })
  const dogPostCid = await ipfs.dag.put({
    content: "dogs",
    author: samCid,
    tags: ["funny", "hobby"],
    prev: computerPostCid
  })

  const outdoorTagCid = await ipfs.dag.put({
    tag: "outdoor",
    posts: [treePostCid]
  })
  const hobbyTagCid = await ipfs.dag.put({
    tag: "hobby",
    posts: [treePostCid, computerPostCid, dogPostCid]
  })
  const funnyTagCid = await ipfs.dag.put({
    tag: "funny",
    posts: [dogPostCid]
  })

  return traversePosts(dogPostCid)
}

return run
`

const solution = `/* globals ipfs */

const traversePosts = async (cid) => {
  const result = []
  while (cid) {
    result.push(cid)
    const current = await ipfs.dag.get(cid)
    const prev = current.value.prev
    if (prev) {
      cid = prev
    } else {
      return result
    }
  }
}

const run = async () => {
  const natCid = await ipfs.dag.put({ author: "Nat" })
  const samCid = await ipfs.dag.put({ author: "Sam" })

  const treePostCid = await ipfs.dag.put({
    content: "trees",
    author: samCid,
    tags: ["outdoor", "hobby"]
  })
  const computerPostCid = await ipfs.dag.put({
    content: "computers",
    author: natCid,
    tags: ["hobby"],
    prev: treePostCid
  })
  const dogPostCid = await ipfs.dag.put({
    content: "dogs",
    author: samCid,
    tags: ["funny", "hobby"],
    prev: computerPostCid
  })

  const outdoorTagCid = await ipfs.dag.put({
    tag: "outdoor",
    posts: [treePostCid]
  })
  const hobbyTagCid = await ipfs.dag.put({
    tag: "hobby",
    posts: [treePostCid, computerPostCid, dogPostCid]
  })
  const funnyTagCid = await ipfs.dag.put({
    tag: "funny",
    posts: [dogPostCid]
  })

  return traversePosts(dogPostCid)
}

return run
`

let modules = { cids: require('cids') }

export default {
  components: {
    Lesson
  },
  data: () => {
    return { code, text, validate, modules, exercise, solution }
  }
}
</script>
